This is an unofficial patch for Carry On 2.2.4.4 on NeoForge 1.21.1.

<img width="1254" height="1254" alt="CarryonYankeeEdition" src="https://github.com/user-attachments/assets/29ecd5af-2fee-41af-b3a7-545f65effc44" />


# Carry On Patched

Unofficial patch for **Carry On 2.2.4.4** on **NeoForge 1.21.1**.

---

## What this patch does

The entire body of `potionLevel` was replaced with a minimal 2-instruction stub:

```java
0: iconst_0
1: ireturn
```

That is the complete patched method.

It takes its two arguments, ignores them, and unconditionally returns `0`.

The patched method contains:

* No branches
* No field reads
* No method calls
* No NBT access of any kind

Because of this, the method is structurally incapable of triggering the malformed NBT crash path.

---

## Why returning `0` fixes the issue

Examining the single call site in `onCarryTick`, the integer returned by `potionLevel` is passed as the duration parameter of:

```java
MobEffectInstance(
    Slowness,
    duration,
    amplifier = 0,
    ambient = false,
    visible = false
)
```

Returning `0` produces:

```text
0-tick duration → effect expires immediately → no slowness applied
```

The amplifier was already hardcoded to `0` in the caller and was never influenced by `potionLevel`'s return value, despite the method name suggesting otherwise.

As a result, returning `0` creates the cleanest operational behavior:

* Players are not slowed while carrying blocks/entities
* No malformed NBT is ever read by this method
* The crash path is removed entirely

---

## Notes about behavior changes

This patched version does **not** calculate the slowness debuff.

That tradeoff is intentional.

Compared to partial `try/catch` patches, this approach is operationally safer because:

* No malformed NBT is ever touched
* No logging path can retrigger the same crash
* No repeated exception spam occurs every tick
* No hidden performance cost from exception handling loops
* The dangerous execution path is removed entirely

---

## Scope of the patch

Exactly one method in one class was modified.

Nothing else in the jar was changed, including:

* `mods.toml`
* manifests
* services
* resource files
* other classes
