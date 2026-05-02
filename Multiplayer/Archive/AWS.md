# Ovweview

Using the AWS SDK for Unreal Engine to communicate with AWS


List of aws cpp sdk libs
```
.\vcpkg search aws-sdk-cpp
```


Install commands

- All
```
.\vcpkg install aws-sdk-cpp[*]:x64-windows --recurse
```
- Specific
```
.\vcpkg install aws-sdk-cpp[core,cognito-idp,apigateway,cloudformation,gamelift,lambda,s3]:x64-windows --recurse
```


# Creating the AWS SDK Plugin

[How to Integrate the AWS C++ SDK with Unreal Engine](https://aws.amazon.com/blogs/gametech/how-to-integrate-the-aws-c-sdk-with-unreal-engine/)


1. Clone https://github.com/microsoft/vcpkg
2. Run Script
    ```
    .\vcpkg\bootstrap-vcpkg.bat
    ```
3. Navigate to the vcpkg directory and run this command to install packages with vcpkg: 

    ```
    .\vcpkg install aws-sdk-cpp[core,kinesis,cognito-identity]:x64-windows --recurse
    ```

    Put whatever service SDKs you want to install in the [ brackets ]. Make sure to at least install the SDK Core (ie “core”). Additionally, don’t forget to specify the platform you’re on. x64-windows/x64-linux/x64-osx for 64-bit Windows, Linux, and MacOS respectively. 

    The above command will install the specific service SDKs you need as well as any dependencies. After it’s finished running (this will take a few minutes) go into vcpkg/installed/[platform] and take note of these folders: 

    - bin/ folder has the dynamic libraries (.dll files)
    - lib/ has the static libraries (.lib files)
    - include/ folder has the header files (.h files)

4. Creating a Plugin from the SDK

    - Open your Unreal Engine project and open Edit > Plugins on the top left. Create a new blank plugin and name it whatever you like (I named it AWSPlugin below).

    - Navigate to the newly created Plugin folder. Under Source/ create a new folder to house the AWS SDK. Again, name this whatever you like, but keep the name distinct from the plugin name (e.g. AWSSDK). This will be our SDK module folder. (If you do not see the source folder, then your project is a blueprint-only project or you’ve created a content-only plugin. This will only work for C++ projects)

    - Add folders for libraries and the header files. I’ve named them Binaries/Win64/ and Include/ respectively. Add a file called [MODULE_NAME].Build.cs. The module name is the same as the folder name (in this case AWSSDK).

    - Copy over the header files from the vcpkg include/ folder earlier into the new include/ folder. This will provide a central location for your code to reference header files.

    - Put static and dynamic libraries (.lib and .dll files) in your platform’s folder under Binaries/. If you remember, these files will come from the bin/ and lib/ folders from the vcpkg installation.

    - Fill your [MODULE_NAME].Build.cs file with code similar to below:

        ```
        using System;
        using System.Collections.Generic;
        using System.IO;
        using UnrealBuildTool;

        public class AWSSDK : ModuleRules
        {
            // list every library you plan to use here
            private List<string> LibraryNames = new List<string>()
            {
                "aws-c-auth",
                "aws-c-cal",
                "aws-c-common",
                "aws-c-compression",
                "aws-c-event-stream",
                "aws-checksums",
                "aws-c-http",
                "aws-c-io",
                "aws-c-mqtt",
                "aws-cpp-sdk-cognito-identity",
                "aws-cpp-sdk-cognito-idp",
                "aws-cpp-sdk-core",
                "aws-crt-cpp",
                "aws-c-s3",
                "aws-c-sdkutils"
            };

            public AWSSDK(ReadOnlyTargetRules Target) : base(Target)
            {
                // ModuleType.External tells the engine not to look for (or compile) any source code.
                Type = ModuleType.External;
                PublicIncludePaths.Add(Path.Combine(ModuleDirectory, "Include"));

                PublicDefinitions.Add("USE_IMPORT_EXPORT");

                if (Target.Type == TargetRules.TargetType.Editor || Target.Type == TargetRules.TargetType.Client)
                {
                    PublicDefinitions.Add("AWS_USE_IO_COMPLETION_PORTS=1");
                    PublicDefinitions.Add("__clang_analyzer__=0");
                    PublicDefinitions.Add("AWS_DEEP_CHECKS=0");
                    PublicDefinitions.Add("_LIBCPP_STD_VER=20");
                    string LibrarysPath = Path.Combine(ModuleDirectory, "Binaries", UnrealTargetPlatform.Win64.ToString());
                    string OutputDir = Path.Combine("$(ProjectDir)", "Binaries", UnrealTargetPlatform.Win64.ToString());

                    foreach (string LibName in LibraryNames)
                    {
                        string LibFileName = LibName + ".lib";
                        string DllFileName = LibName + ".dll";

                        // Add the import library
                        PublicAdditionalLibraries.Add(Path.Combine(LibrarysPath, LibFileName));

                        // Do not load the DLL during startup. Delay-load the DLL when it is actually used.
                        PublicDelayLoadDLLs.Add(DllFileName);

                        // This stages the DLL next to the executable when you package your game.
                        RuntimeDependencies.Add(Path.Combine(OutputDir, DllFileName), Path.Combine(LibrarysPath, DllFileName));
                    }
                }
            }
        }
        ```

    - Navigate to your Project’s Build.cs file (should be in a location like: [ProjectName]\Source\[ProjectName]\[ProjectName].Build.cs). Add the AWS SDK Module you created earlier as a dependency (You will see below I added “AWSSDK” in the list) and add the bEnableUndefinedIdentifierWarnings = false; line.

        ```
        PublicDependencyModuleNames.AddRange(new string[] { "Core", "CoreUObject", "Engine", "InputCore", "AWSSDK" });
        //...
        bEnableUndefinedIdentifierWarnings = false;
        ```

    - (bEnableUndefinedIdentifierWarnings = false is necessary to get the build working. The AWS C++ SDK naturally generates warnings which will cause Unreal to fail builds, but that’s nothing to worry about)

    - Navigate back to your project’s root directory and right click the .uproject file. Click on Generate Visual Studio project files and wait for it to finish. Once that is completed, open Visual Studio and click Reload All to register the new files in the editor.

    - If you did everything correctly, it will compile successfully in both Visual Studio and Unreal Engine.


# Start and Stop

```
#include <aws/core/Aws.h>

Aws::SDKOptions options;
Aws::InitAPI(options); // <--- Must do this first

Aws::ShutdownAPI(options); // <--- Must do this before exiting
```

# Creating the AWS Subsystem for Unreal Engine

In Unreal Engine, a Subsystem is a managed, modular class that has a controlled lifetime.

Essentially, they provide a way to create global-like objects without the messy overhead of manually managing their creation, initialization, or destruction.


1. Right click on the C++ directory of the Content Browser of UE and select "New C++ Class..."
2. Select "GameInstanceSubsystem"

    Automatically starts when the game start and closes when the game close.

3. Override these functions in the header file:

    ```
    public:
        // Automatically called by UE when the game starts/launches
        virtual void Initialize(FSubsystemCollectionBase& Collection) override;

        // Automatically called by UE when the game closes
        virtual void Deinitialize() override;
    ```

4. Define them in the cpp file

    ```
    void UAwsSubsystem::Initialize(FSubsystemCollectionBase& Collection)
    {
        Super::Initialize(Collection);

        UE_LOG(LogTemp, Display, TEXT("Subsystem is starting..."));
    }

    void UAwsSubsystem::Deinitialize()
    {
        UE_LOG(LogTemp, Display, TEXT("Subsystem is stopping..."));

        Super::Deinitialize();
    }
    ```

5. Now you can Start / Stop the AWS SDK

    - In the header file, declare the options variable

        ```
        #include <aws/core/Aws.h>
        
        // Stored to ensure ShutdownAPI uses the same configuration as InitAPI
        Aws::SDKOptions Options;
        ```
    - In the C++ file, call InitAPI and ShutdownAPI 

        ```
        void UAwsSubsystem::Initialize(FSubsystemCollectionBase& Collection)
        {
            Super::Initialize(Collection);

            ...
            Aws::InitAPI(Options);
            ...
        }

        void UAwsSubsystem::Deinitialize()
        {
            ...
            Aws::ShutdownAPI(Options);
            ...
            Super::Deinitialize();
        }
        ```

6. Creating the Config and Cient

- Header

    ```
    #include <aws/core/Aws.h>
    #include <aws/cognito-idp/CognitoIdentityProviderClient.h>
    #include "aws/cognito-idp/model/ResendConfirmationCodeRequest.h"


    protected:
        Aws::SDKOptions Options;
        // Use a unique_ptr or shared_ptr for the config to delay its creation
        std::unique_ptr<Aws::Client::ClientConfiguration> ClientConfig;
        std::shared_ptr<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient> CognitoClient;
    ```
- CPP
    ```
    void UAwsSubsystem::Initialize(FSubsystemCollectionBase& Collection)
    {
        Super::Initialize(Collection);
        
        UE_LOG(LogTemp, Display, TEXT("Subsystem is starting..."));

        ...


        ClientConfig = std::make_unique<Aws::Client::ClientConfiguration>();
        ClientConfig->region = "us-east-1";

        CognitoClient = std::make_shared<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient>(*ClientConfig);


    }

    void UMMOOnlineSubsystem::Deinitialize()
    {
        UE_LOG(LogTemp, Display, TEXT("Subsystem is stopping..."));

        CognitoClient.reset();
        Aws::ShutdownAPI(Options);

        Super::Deinitialize();
    }

    ```

## Defining and using Blueprint callable functions

1. In the subsystem header file, declare a blueprint callable function

    ```
    UFUNCTION(BlueprintCallable, Category = "GameLogic")
    void ExampleFunction(float Volume);
    ```

2. In the C++ file, define the function

    ```
    void UTestSubsystem:ExampleFunction(float Volume)
    {
        UE_LOG(LogTemp, Display, TEXT("Blueprint callable function called"));

    }
    ```

3. In the blueprint, get the subsystem and call the function

    - Call "Get your-sub-system-name"
    - Call your blueprint callable function
    
    ![](./AWS%20Gamelift/TestSubsystem.png)

## Calling Subsystem functions in C++

```
#include <TestSubsystem.h>

...

// Inside any Actor or Widget
UTestSubsystem* Sub = GetGameInstance()->GetSubsystem<UTestSubsystem>();
if (Sub)
{
    Sub->ExampleFunction(1.0f);
}
```

## Defining Configurations using INI files


1. Add the Config specifier to the UCLASS Macro

    ```
    UCLASS(Config = Game) // Loads from DefaultGame.ini
    ```
2. Mark variables with UPROPERTY(Config)

    ```
    UPROPERTY(Config)
    float MovementSpeedMultiplier;
    ```

3. In your DefaultGame.ini:

    ```
    [/Script/YourProjectName.MySubsystem]
    MovementSpeedMultiplier=1.5
    ```

## Defining Configurations Project Settings

- Dependencies

    ````
    PrivateDependencyModuleNames.AddRange(
	new string[]
	{
		...
        "DeveloperSettings",
	}
	);
    ```

1. Create a separate class inheriting from UDeveloperSettings.


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

2. Accessing the configuration

    ```
    // Inside your Subsystem.cpp
    void UMySubsystem::Initialize(FSubsystemCollectionBase& Collection) {
        const UMySettings* Settings = GetDefault<UMySettings>();
        int32 Retries = Settings->MaxRetryAttempts;
    }
    ```