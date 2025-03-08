# Lambda: An Example Language: Step 2 Spec

This is the part of the spec that should pass after completing step 1 of this
tutorial.

Language: `lambda`

## Step 1

```haskell
foo = bar;
```
```status
ERROR: bar is not a valid lambda expression in foo = bar;
```

## Step 2

```haskell
id = \x. x;
```
```status
Success
```

```haskell
id = \x. y;
```
```status
ERROR: y is not a valid lambda expression in id = \x. y;
```
