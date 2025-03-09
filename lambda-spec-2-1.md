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
