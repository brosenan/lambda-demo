# Lambda: An Example Language

This is a shortened spec for the `lambda` language.

Language: `lambda`

## Step 1

```haskell
foo = bar;
```
```status
ERROR: bar is not a valid lambda expression in foo = bar;
```

## Step 2.1

```haskell
id = \x. x;
```
```status
Success
```

## Step 2.2

```haskell
id = \x. y;
```
```status
ERROR: y is not a valid lambda expression in id = \x. y;
```

## Step 3.1

```haskell
one = \f.\x.f x;
```
```status
Success
```

## Step 3.2

```haskell
one = \f.\x.f1 x;
```
```status
ERROR: f1 is not a valid lambda expression in one = \f.\x.f1 x;
```

## Step 3.3

```haskell
one = \f.\x.f x1;
```
```status
ERROR: x1 is not a valid lambda expression in one = \f.\x.f x1;
```

## Step 3.4

```haskell
PLUS = \m.\n.\f.\x.m f (n f x);
```
```status
Success
```

## Step 4.1

```haskell
zero = \f.\x.x;
PLUS = \m.\n.\f.\x.m f (n f x);
MULT = \m.\n.m (PLUS n) zero;
```
```status
Success
```

## Step 4.2

```haskell
-- The identity function
id = \x. x;

-- Church Numerals
zero = \f.\x.x;
one = \f.\x.f x;
two = \f.\x.f (f x);

-- Arithmetic
PLUS = \m.\n.\f.\x.m f (n f x);
MULT = \m.\n.m (PLUS n) zero;

-- Logic
TRUE = \x.\y.x;
FALSE = \x.\y.y;
AND = \p.\q.p q p;
OR = \p.\q.p p q;
NOT = \p.p FALSE TRUE;
IFTHENELSE = \p.\a.\b.p a b;
```
```status
Success
```

## Step 4.3

```haskell
-- The identity function
id = \x. x;

-- Church Numerals
zero = \f x.x;
one = \f x.f x;
two = \f x.f (f x);

-- Arithmetic
PLUS = \m n f x.m f (n f x);
MULT = \m n.m (PLUS n) zero;

-- Logic
TRUE = \x y.x;
FALSE = \x y.y;
AND = \p q.p q p;
OR = \p q.p p q;
NOT = \p.p FALSE TRUE;
IFTHENELSE = \p a b.p a b;
```
```status
Success
```
