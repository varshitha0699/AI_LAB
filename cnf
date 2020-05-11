import re

class Expr:
    def __init__(self, op, *args):
        self.op = op
        self.args = [expr(arg) for arg in args] # Coerce args to Exprs

    def __repr__(self):
        if len(self.args)==0:         # Constant or proposition with arity 0
            return str(self.op)
        elif len(self.args) == 1: # Prefix operator
            return self.op + repr(self.args[0])
        else:                     # Infix operator
            return '(%s)' % (' '+self.op+' ').join([repr(arg) for arg in self.args])
        
    def __and__(self, other):    return Expr('&',  self, other)
    def __invert__(self):        return Expr('~',  self)
    def __rshift__(self, other): return Expr('>>', self, other)
    def __or__(self, other):     return Expr('|',  self, other)
    def __mod__(self, other):    return Expr('<=>',  self, other)

def expr(s):
    if isinstance(s, Expr): return s
    if is_symbol(s): return Expr(s)
    s = s.replace('==>','>>')
    s = s.replace('<=>','%')
    s = re.sub(r'([a-zA-Z0-9_.]+)', r'Expr("\1")', s)
    # Now eval the string.
    return eval(s, {'Expr':Expr})

def is_symbol(s):
    "A string s is a symbol if it starts with an alphabetic char."
    return isinstance(s, str) and s[0].isalpha() and len(s) == 1

def find_if(predicate, seq):
    "If there is an element of seq that satisfies predicate; return it"
    for x in seq:
        if predicate(x): return x
    return None

def eliminate_implications(s):
    if not s.args or is_symbol(s.op): return s     ## (Atoms are unchanged.)
    args = [eliminate_implications(arg) for arg in s.args]
    a, b = args[0], args[-1]
    if s.op == '>>':
         return (~a | b)
    elif s.op == '<=>':
        return (~a | b) & (~b | a)
    else:
        return Expr(s.op, *args)

def move_not_inwards(s):
    if s.op == '~':
        NOT = lambda b: move_not_inwards(~b)
        a = s.args[0]
        if a.op == '~': return move_not_inwards(a.args[0]) # ~~A ==> A
        if a.op =='&': return associate('|', map(NOT, a.args))
        if a.op =='|': return associate('&', map(NOT, a.args))
        return s
    elif s.op or not s.args:
        return s
    else:
        return Expr(s.op, *[move_not_inwards(arg) for arg in s.args])

def distribute_and_over_or(s):
    if s.op == '|':
        s = associate('|', s.args)
        if s.op != '|':
            return distribute_and_over_or(s)
        if len(list(s.args)) == 0:
            return FALSE
        if len(list(s.args)) == 1:
            return distribute_and_over_or(s.args[0])
        conj = find_if((lambda d: d.op == '&'), s.args)
        if not conj:
            return s
        others = [a for a in s.args if a is not conj]
        rest = associate('|', others)
        return associate('&', [distribute_and_over_or(c|rest)
                               for c in conj.args])
    elif s.op == '&':
        return associate('&', [distribute_and_over_or(arg) for arg in s.args])
    else:
        return s

def associate(op, args):
    args = dissociate(op, args)
    if len(args) == 0:
        return _op_identity[op]
    elif len(args) == 1:
        return args[0]
    else:
        return Expr(op, *args)

def dissociate(op, args):
    result = []
    def collect(subargs):
        for arg in subargs:
            if arg.op == op: collect(arg.args)
            else: result.append(arg)
    collect(args)
    return result

def conjuncts(s):
    return dissociate('&', [s])

def disjuncts(s):
    return dissociate('|', [s])

def to_cnf(s):
    if isinstance(s, str):s = expr(s)
    s = eliminate_implications(s) # Eliminate Conditional and Biconditional
    print(f'Eliminated Implications : {s}')
    s = move_not_inwards(s) # Demorgan's law
    print(f'After DeMorgans Law : {s}')
    s = distribute_and_over_or(s) # Distributitvity law
    print(f'After Distributivity Law : {s}')
    print(f'CNF : {s}')


TRUE, FALSE, ZERO, ONE, TWO = map(Expr, ['TRUE', 'FALSE', 0, 1, 2])
_op_identity = {'&':TRUE, '|':FALSE, '+':ZERO, '*':ONE}

print("AND : & ; OR: | ; NOT: ~ ; IMPLIES : ==> ; EQUIVALENCE : <=>")
prop=input("Enter the propositional logic  :  ")
to_cnf(prop)
