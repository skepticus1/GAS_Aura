# GAS Notes:

## Game Startup Order (Simplified)
- **GameMode** → Controls rules, player spawning, and overall game flow.
- **PlayerController** → Manages player input and interactions.
- **PlayerState** → Stores player-specific data (ASC, Attributes, Score, etc.).
- **Character/Pawn** → The actual in-game character that the player controls.
- **Components (ASC, AttributeSet, etc.)** → Created in constructors of PlayerState/Character.
- **HUD/UI** → Displays game info to the player.


## Game Flow (Step by Step)

1. **GameMode set in Project Settings.** Set to `BP_AuraGameMode`.

2. **`BP_AuraGameMode` is created.**
   - Defines the `PlayerState`, `PlayerController`, `Character`, `HUD`, etc.
   - Each player gets a `BP_AuraPlayerState` and a `BP_AuraPlayerController`.

3. **The GameMode creates an instance of `BP_AuraPlayerController`.**

4. **The GameMode creates an instance of `BP_AuraPlayerState`.**  
   This will create the `AbilitySystemComponent` and the `AttributeSet` from `AuraAbilitySystemComponent` and `AuraAbilitySet` in its constructor.
```cpp
	AAuraPlayerState::AAuraPlayerState()
	{
		AbilitySystemComponent = CreateDefaultSubobject<UAuraAbilitySystemComponent>("AbilitySystemComponent");
		AttributeSet = CreateDefaultSubobject<UAuraAttributeSet>("AttributeSet");
	}
```

5. **The GameMode creates an instance of `BP_AuraCharacter` and spawns it.**

6. **The instance of the `BP_AuraPlayerController` possesses the instance of the `BP_AuraCharacter`.**  
The `PlayerController` possesses the `Pawn` with code in the `APlayerController` class.  
The default pawn from the game mode is how it knows.  
This code in the `PlayerController` gets the controlled `Pawn` to apply controls to it.
```cpp	
	if (APawn* ControlledPawn = GetPawn<APawn>()) {
		ControlledPawn->AddMovementInput(ForwardDirection, InputAxisVector.Y);
		ControlledPawn->AddMovementInput(RightDirection, InputAxisVector.X);
	}
```

7. **The GameMode creates the `AAuraHUD` from the `BP_AuraHUD`.**  
Unreal Engine automatically creates an instance when the game starts.

- **`InitOverlay()` connects the `AuraHUD` to GAS and UI.**
```cpp	
		void AAuraHUD::InitOverlay(APlayerController* PC, APlayerState* PS, UAbilitySystemComponent* ASC, UAttributeSet* AS)
		{		
			checkf(OverlayWidgetClass, TEXT("Overlay Widget Class uninitialized, please fill out BP_AuraHUD"));
			checkf(OverlayWidgetControllerClass, TEXT("Overlay Widget Controller Class uninitialized, please fill out BP_AuraHUD"));

			UUserWidget* Widget = CreateWidget<UUserWidget>(GetWorld(), OverlayWidgetClass);
			OverlayWidget = Cast<UAuraUserWidget>(Widget);

			const FWidgetControllerParams WidgetControllerParams(PC, PS, ASC, AS);
			UOverlayWidgetController* WidgetController = GetOverlayWidgetController(WidgetControllerParams);
			OverlayWidget->SetWidgetController(WidgetController);

			Widget->AddToViewport();
		}
```
**Connections in Simple Steps**
- ✅ GameMode → Spawns `AuraHUD` automatically.
- ✅ `AuraHUD::InitOverlay()` → Links `PlayerController`, `PlayerState`, `ASC`, and `Attributes` to the UI.
- ✅ `AuraHUD` → Creates `AuraUserWidget` and assigns an `OverlayWidgetController` to manage UI updates.
- ✅ `OverlayWidgetController` → Acts as a middleman between the UI and GAS.

---


## Helpful Notes: Accessing `PlayerState` in Code

### **From `PlayerController`:**
```cpp
		APlayerState* PS = GetPlayerState<APlayerState>();
```
This returns BP_AuraPlayerState because GameMode set it.
    
### **From `Character`:**
```cpp
    APlayerState* PS = GetPlayerState<APlayerState>();
```
This retrieves the PlayerState linked to the Character.

### **From `PlayerState` (to get `Controller` or `Character`):**
```cpp
		AController* PC = GetOwner<AController>();
		APawn* MyPawn = GetPawn();
```
