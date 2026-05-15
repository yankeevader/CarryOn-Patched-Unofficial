This is an unofficial patch for Carry On 2.2.4.4 on NeoForge 1.21.1.

What the patch does
The entire body of potionLevel was replaced with a 2-instruction stub:

0: iconst_0    // push the integer 0 onto the stack
1: ireturn     // return it
That is the entire patched method. It takes its two arguments, ignores them both, and unconditionally returns 0. There are no branches, no field reads, no method calls, no NBT access of any kind. It is structurally incapable of throwing any exception.

Why returning 0 specifically
Examining the single call site in onCarryTick, the int returned by potionLevel is passed as the duration parameter (in ticks) of a MobEffectInstance(Slowness, duration, amplifier=0, ambient=false, visible=false) that gets applied to the carrying player.

Returning 0 → 0-tick duration → effect expires the same tick it's applied → no slowness applied to the player at all.
The amplifier was hardcoded to 0 in the caller and was never affected by potionLevel's return value, despite the method's misleading name.
So returning 0 produces the cleanest possible behavior: the player is not slowed while carrying anything, and no NBT is ever read by this method.

This Patched Version Does Not Calculate Slowness Debuff - amplifier was already hardcoded to 0 anyways
-- NOTE --
arguably safer operationally than partial try/catch patches because:

no malformed NBT ever gets touched
no logging path can retrigger the same crash
no repeated exception spam every tick
no hidden performance hit from exception handling loops
removed the dangerous execution path entirely

Exactly one method in one class was modified. Nothing else in the jar — manifests, mods.toml, services, resource files, other classes — was touched.
