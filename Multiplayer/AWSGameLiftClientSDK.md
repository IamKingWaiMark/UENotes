# OVerview

Using the GameLiftClientSDK


1. Add the GameLiftClientSDK module to your project (Project.Build.cs)

    ```
    PublicDependencyModuleNames.AddRange(new string[] {
        ...
        "GameLiftClientSDK"
    });

    PublicDefinitions.Add("USE_IMPORT_EXPORT");

    if (Target.Type == TargetRules.TargetType.Editor || Target.Type == TargetRules.TargetType.Client)
    {
        PublicDependencyModuleNames.AddRange(new string[] { "AWSSDK", "CoreSDK" });

        PublicDefinitions.Add("WITH_GAMELIFT_CLIENT=1");
    }
    else
    {
        PublicDefinitions.Add("WITH_GAMELIFT_CLIENT=0");
    }
    ```


2. Create Custom Settings

    - Add DeveloperSettings module

        ```
        PrivateDependencyModuleNames.AddRange(
        new string[]
        {
            ...
            "DeveloperSettings"
        }
        );
        ```

    - Create a separate class inheriting from UDeveloperSettings.

        ```
        // MySettings.h
        UCLASS(Config = Game, defaultconfig, meta = (DisplayName = "My Subsystem Settings"))
        class UMySettings : public UDeveloperSettings {
            GENERATED_BODY()

        public:
            UPROPERTY(Config, EditAnywhere, Category = "General")
            int32 MaxRetryAttempts;
        };
        ```

    - Accessing the configuration

        ```
        // Inside your Subsystem.cpp
        void UMySubsystem::Initialize(FSubsystemCollectionBase& Collection) {
            const UMySettings* Settings = GetDefault<UMySettings>();
            int32 Retries = Settings->MaxRetryAttempts;
        }
        ```