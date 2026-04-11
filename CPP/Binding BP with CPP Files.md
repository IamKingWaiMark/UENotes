# Overview

Using CPP files for your Blueprint files


# Binding UserWidget BP with a UserWidget CPP File


1. Create a UserWidget Blueprint (e.g. BP_MyUI)
2. Open the UserWidget Blueprint and create the UI
    - Add Canvas
    - Add Button (Name: Button_0)
3. Create a UserWidget CPP File in the C++ Classes directory
    - Right-click in the content browser of the C++ Classes directory > New C++ Class...
    - Click on "All Classes" tab and search for "User Widget"

        ![](./Assets/Binding%20BP%20with%20CPP%20Files/UserWidgetCPP.png)

4. Open the UserWidget blueprint (BP_MyUI) and switch to the "Graph" view
    - Click on "Class Settings"
    - In the "Details" panel, set the "Parent Class" to the CPP Widget

        ![](./Assets/Binding%20BP%20with%20CPP%20Files/BindCPPWithBlueprint.png)

## Binding UI Components with Code (Button)

1. Open the header file for the CPP UserWidget (e.g.  MyUI.h)
2. Import the Button class
    ```
    #include "Components/Button.h"
    ```
3. Define the Constructor, button, and a function for the button click action

    ```
    protected:

	// Standard way to initialize logic
	virtual void NativeConstruct() override;

	// This looks for a Button named "Button_9" in your Blueprint
	UPROPERTY(meta = (BindWidget))
	class UButton* Button_0;
	
    // Function for when the button is clicked
	UFUNCTION()
	void OnSubmitClicked();
    ```
    
4. Open the cpp file for the UserWidget and define the logic

    ```
    void UMyUI::NativeConstruct()
    {
        Super::NativeConstruct();

        if (Button_0)
        {
            // Call the OnSubmitClicked function when the button is clicked
            Button_0->OnClicked.AddDynamic(this, &UMyUI::OnSubmitClicked);

        }
    }

    void UMyUI::OnSubmitClicked()
    {
        // Your logic here!
        UE_LOG(LogTemp, Warning, TEXT("Button was clicked in C++!"));
    }
    ```

