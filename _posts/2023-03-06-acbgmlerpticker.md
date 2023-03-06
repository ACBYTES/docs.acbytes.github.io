---
title: Lerp Ticker
date: 2023-03-06 01:42:00 -0500
categories: [Regret, Interpolation]
tags: [unreal,ue4,ue5,interp,interpolation,lerper,c++]
---

# What is Lerp Ticker?

`Lerp Ticker` is the class that is responsible with handling global tick function calls for the [Lerper](../lerper) and [DynamicLerper](../dynamiclerper) objects.

Lerper ticker is a `UActorComponent`, attached to an actor, created in the main `GameMode` so that each level has `Lerp Ticker` instantiated.

```cpp
#pragma once

#include "CoreMinimal.h"
#include "Components/ActorComponent.h"
#include "ACBGMLerpTicker.generated.h"

DECLARE_MULTICAST_DELEGATE_OneParam(FOnUniversalTick, float)

UCLASS( ClassGroup=(Custom), meta=(BlueprintSpawnableComponent) )
class REGRET_API UACBGMLerpTicker final : public UActorComponent
{
	GENERATED_BODY()

public:	
	UACBGMLerpTicker();
	virtual ~UACBGMLerpTicker() override;

	template <typename T>
	static FDelegateHandle BindTickFunction(T* ClassPtr, void(T::*Func)(float))
	{
		return Active->OnUniversalTick.AddRaw(ClassPtr, Func);
	}
	
	static void UnbindTickFunction(const FDelegateHandle& DelegateHandle);

protected:
	FOnUniversalTick OnUniversalTick;
	static UACBGMLerpTicker* Active;

private:
	virtual void OnComponentCreated() override;
	virtual void TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction) override;
};
```

`FOnUniversalTick` is declared using Unreal Engine's `DECLARE_MULTICAST_DELEGATE_OneParam` macro. Based on that, `OnUniversalTick` will be declared and referenced by the class.

> `BindTickFunction` is now implicitly `inline`d. Declaration and implementation of template types is only possible in the header file.
{: .prompt-tip}

```cpp
#include "ACBGMLerpTicker.h"

UACBGMLerpTicker* UACBGMLerpTicker::Active;

UACBGMLerpTicker::UACBGMLerpTicker()
{
	PrimaryComponentTick.bCanEverTick = true;
}

UACBGMLerpTicker::~UACBGMLerpTicker()
{
}

void UACBGMLerpTicker::OnComponentCreated()
{
	Active = this;
}

void UACBGMLerpTicker::TickComponent(float DeltaTime, ELevelTick TickType, FActorComponentTickFunction* ThisTickFunction)
{
	Super::TickComponent(DeltaTime, TickType, ThisTickFunction);
	OnUniversalTick.Broadcast(DeltaTime);
}

void UACBGMLerpTicker::UnbindTickFunction(const FDelegateHandle& DelegateHandle)
{
	Active->SetComponentTickEnabled(false);
	Active->OnUniversalTick.Remove(DelegateHandle);
	Active->SetComponentTickEnabled(true);
}
```

We use `OnComponentCreated` to set our singleton `Active` to `this`.

In `TickComponent`, we broadcast our delegate to all the bound functions.

In `UnbindTickFunction`, we temporarily disable `Active`'s tick function. Then, we remove the bound function from `UniversalTick`. Finally, `Active`'s tick function gets re-enabled.

# Why was `OnComponentCreated` used?

The question that might arise is that why `OnComponentCreated` has been used to assign the singleton, and not the constructor itself. The ticker class is a `UObject`
and it's instantiated is the engine's responsibility. The behavior is not guaranteed and as a result, using `OnComponentCreated` will be the guaranteed solution to our singleton problem (While debugging, it seems that after the instantiation of any `UObject`, the object will not be in the same memory address as it used to be during its constructor. It's possible that the move constructor gets called, as no reconstruction happens for us to assign `Active` to the proper object in memory.).