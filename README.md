[![](https://img.shields.io/nuget/v/soenneker.atomics.valuenullablebools.svg?style=for-the-badge)](https://www.nuget.org/packages/soenneker.atomics.valuenullablebools/)
[![](https://img.shields.io/github/actions/workflow/status/soenneker/soenneker.atomics.valuenullablebools/publish-package.yml?style=for-the-badge)](https://github.com/soenneker/soenneker.atomics.valuenullablebools/actions/workflows/publish-package.yml)
[![](https://img.shields.io/nuget/dt/soenneker.atomics.valuenullablebools.svg?style=for-the-badge)](https://www.nuget.org/packages/soenneker.atomics.valuenullablebools/)
[![](https://img.shields.io/github/actions/workflow/status/soenneker/soenneker.atomics.valuenullablebools/codeql.yml?label=CodeQL&style=for-the-badge)](https://github.com/soenneker/soenneker.atomics.valuenullablebools/actions/workflows/codeql.yml)

# Soenneker.Atomics.ValueNullableBools

A lightweight, allocation-free atomic tri-state flag implemented on top of an inline `ValueAtomicInt`. Backing values: `-1` = null / unknown `0` = false `1` = true.

## Install

```bash
dotnet add package Soenneker.Atomics.ValueNullableBools
```

## Usage

```csharp
using Soenneker.Atomics.ValueNullableBools;

public sealed class FeatureProbe
{
    private ValueAtomicNullableBool _supported;

    public bool Publish(bool result) => _supported.TrySet(result);
    public bool? Read() => _supported.Value;
    public void Invalidate() => _supported.Reset();
}
```

The default value represents `null`/unknown. `TrySet` atomically lets one caller publish the first known boolean. `GetValueOrFalse` and `GetValueOrTrue` read with an explicit fallback without changing the stored state.

This is a mutable struct and must remain in stable field storage. Passing it by value or returning it from a property creates independent tri-state state. Use the reference-type `AtomicNullableBool` when the wrapper itself must be shared.

Prefer the nullable API in application code. Raw `Write` and `TryCompareExchange` accept integer states without validation; only `-1`, `0`, and `1` have defined meaning.

## What you get

- `ValueAtomicNullableBool` — A lightweight, allocation-free atomic tri-state flag implemented on top of an inline `ValueAtomicInt`. Backing values: `-1` = null / unknown `0` = false `1` = true.

## API at a glance

| API | What it does | Result / important behavior |
| --- | --- | --- |
| `ValueAtomicNullableBool.HasValue` | Gets a value indicating whether the current state is non-null. | Gets a value indicating whether the current state is non-null. |
| `ValueAtomicNullableBool.Value` | Gets the current value as a nullable boolean. | Gets the current value as a nullable boolean. |
| `ValueAtomicNullableBool.Read()` | Reads the raw backing state. | `-1` (null), `0` (false), or `1` (true). |
| `ValueAtomicNullableBool.GetValueOrFalse()` | Gets the value, treating `null`/`unknown` as `false`. | true if gets the value, treating null/unknown as; otherwise, false. |
| `ValueAtomicNullableBool.GetValueOrTrue()` | Gets the value, treating `null`/`unknown` as `true`. | true if gets the value, treating null/unknown as; otherwise, false. |
| `ValueAtomicNullableBool.Set(value)` | Sets the state to `true` or `false`. | Returns no value; the requested change is complete when the method returns. |
| `ValueAtomicNullableBool.TrySet(value)` | Attempts to set the state to `true` or `false` only if the current state is `null`/`unknown`. | true if the requested update was applied; otherwise, false. |
| `ValueAtomicNullableBool.TryCompareExchange(newState, expected)` | Attempts to transition the state from `expected` to `newState`. | true if the requested update was applied; otherwise, false. |
| `ValueAtomicNullableBool.Reset()` | Resets the state to `null`/`unknown`. | Returns no value; the requested change is complete when the method returns. |
| `ValueAtomicNullableBool.ToString()` | Returns a string representation of the current state. | Returns `string`. |

## Important behavior

- `ValueAtomicNullableBool`: Reads establish acquire semantics and writes establish release semantics. This is a mutable `struct` intended for use as a private field or inline synchronization primitive. Avoid copying this type or exposing it publicly.
- `ValueAtomicNullableBool.Write(state)`: Callers must only provide valid values: `-1`, `0`, or `1`.
