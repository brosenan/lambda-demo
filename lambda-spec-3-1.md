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
