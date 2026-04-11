# Overview

[Gemini Conversation](https://gemini.google.com/app/d6cd4c4ae349b4ad)

[AWS Gamelift Plugin Guide](https://docs.aws.amazon.com/gameliftservers/latest/developerguide/unreal-plugin.html)


# Setup AWS Account & IAM User

1. Log into [AWS Console](https://aws.amazon.com/console/)
2. Create an [IAM User](https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/home)
    - Create a new user (e.g., GameLift_Dev_User).

        ![](./AWS%20Gamelift/IAM%20Create%20User.png)

        ![](./AWS%20Gamelift/Create%20IAM%20User%202.png)

    - Attach Policies: For development, you can attach <b>AdministratorAccess</b> for ease, but for better security, ensure you at least have <b>AmazonGameLiftFullAccess</b> and <b>AWSCloudFormationFullAccess</b>.

        ![](./AWS%20Gamelift/Create%20IAM%20User%203.png)


3. Generate Access Keys:
    - In the user's details, look for the <b>Access keys</b> section and click on <b>Create access key</b>

        ![](./AWS%20Gamelift/Gen%20Access%20Key%202.png)

    - Select "Application running outside AWS."

        ![](./AWS%20Gamelift/Gen%20Access%20Key%203.png)

    - Important: Save the Access Key ID and Secret Access Key. You will need these to "Bootstrap" the plugin in Unreal.

# Installing and Boostrap the AWS Gamelift Plugin

1. Download the Plugin: Get the latest [Amazon GameLift Plugin for Unreal Engine](https://github.com/amazon-gamelift/amazon-gamelift-plugin-unreal/releases) from GitHub.

    - Select the <b>amazon-gamelift-plugin-unreal-release-x.x.x.zip </b>
2. Install to Project:

    - Navigate to your Unreal Project’s root folder.
    - Create a folder named Plugins if it doesn't exist.
    - Copy the GameLiftPlugin folder into that Plugins directory.

3. Open Unreal & Configure:

    - Open your project. If prompted to rebuild the plugin, click Yes.
    - In the top menu bar, click the new GameLift menu and select AWS Account Settings (or "Set AWS User Profiles").

        ![](./AWS%20Gamelift/AWS%20Plugin%20Tool1.png)

4. Add Your Profile:

    - Click Add Profile.

        ![](./AWS%20Gamelift/AWS%20Plugin%20Tool%202.png)

    - Paste your Access Key ID and Secret Access Key

    - Select your preferred AWS Region (e.g., us-east-1).

5. Bootstrap:

    - Select your profile and click on the "Select profile" button
    - Click the Boothstrap button

        ![](./AWS%20Gamelift/AWS%20Plugin%20Tool%203.png)


# Create Server and Client Build Targets

For an MMO, Unreal needs to know the difference between the Client (what players download) and the Server (the "headless" version that runs on AWS GameLift). By default, Unreal projects only have an "Editor" target.

1. Open your project's Source folder in File Explorer:

    ```
    [ProjectName]/Source/
    ```
2. Find your existing Target file. It will be named something like

    ```
    [ProjectName].Target.cs
    ```
3. Create the Server Target:

    - Duplicate that file and rename the copy to [ProjectName]Server.Target.cs.
    - Open it in a text editor (like Notepad++ or VS Code).
    - Change the class name to [ProjectName]ServerTarget
    - Change TargetType = TargetType.Game; to TargetType = TargetType.Server;


4. Create the Client Target:

    - Duplicate the original file again and rename it to [ProjectName]Client.Target.cs.
    - Open it and change the class name to [ProjectName]ClientTarget.
    - Change the type to TargetType = TargetType.Client;

5. Regenerate Project Files:

    - Go back to your main project folder.
    - Right-click your .uproject file and select "Generate Visual Studio project files".

Why this matters:

A Dedicated Server build for an MMO removes all the "heavy" stuff it doesn't need, like textures, sounds, and user interfaces. This makes the server run much faster and cheaper on AWS.

# Step 4: Modifying the Build.cs and C++ Setup

Now we need to tell Unreal Engine to actually use the GameLift code when it’s building your game. This involves adding the "GameLiftServerSDK" as a dependency.


1. Open your Project’s Module Build File:

    - Navigate to [ProjectName]/Source/[ProjectName]/[ProjectName].Build.cs.

2. Add the GameLift Module:

    - Find the line that starts with PublicDependencyModuleNames.AddRange.
    - Add "GameLiftServerSDK"

        ```
        // Add the following section
        if (Target.Type == TargetType.Server)
        {
            PublicDependencyModuleNames.Add("GameLiftServerSDK");
        }
        else
        {
            PublicDefinitions.Add("WITH_GAMELIFT=0");
        }
        bEnableExceptions =  true;
        ```

3. Why bEnableExceptions = true?

    Standard Unreal C++ usually disables exceptions to save performance. However, the GameLift SDK uses them to handle connection errors. If you don't turn this on, your server build will likely fail to compile.

4. Save and Refresh:

    Save the file, then right-click your .uproject file and Generate Visual Studio project files again to make sure the build system picks up the new "Exceptions" rule.


# Step 5: The "Heartbeat" Code — Initializing GameLift

[Update your game server code](https://docs.aws.amazon.com/gameliftservers/latest/developerguide/unreal-plugin-integrate.html#unreal-plugin-anywhere-integrate-simple-server)

Update your game server code to enable communication between a game server process and the Amazon GameLift Servers service. Your game server must be able to respond to requests from Amazon GameLift Servers, such as to start and stop new game sessions.


1. Example gameMode.h code

```
// Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

#pragma once

#include "CoreMinimal.h"
#include "GameFramework/GameModeBase.h"
#include "GameLiftUnrealAppGameMode.generated.h"

struct FProcessParameters;

DECLARE_LOG_CATEGORY_EXTERN(GameServerLog, Log, All);

UCLASS(minimalapi)
class AGameLiftUnrealAppGameMode : public AGameModeBase
{
    GENERATED_BODY()

public:
    AGameLiftUnrealAppGameMode();

protected:
    virtual void BeginPlay() override;

private:
    void InitGameLift();

private:
    TSharedPtr<FProcessParameters> ProcessParameters;
}; 
```


2. Example game server code

Open the related source file [project-name]GameMode.cpp file (for example GameLiftUnrealAppGameMode.cpp). Change the code to align with the following example code. Be sure to replace "GameLiftUnrealApp" with your own project name. These updates are specific to the game server; we recommend that you make a backup copy of the original file for use with your client.

The following example code shows how to add the minimum required elements for server integration with Amazon GameLift Servers:

    Initialize an Amazon GameLift Servers API client. The InitSDK() call with server parameters is required for an Amazon GameLift Servers Anywhere fleet. When you connect to an Anywhere fleet, the plugin stores the server parameters as console arguments The sample code can access the values at runtime.

    Implement required callback functions to respond to requests from the Amazon GameLift Servers service, including OnStartGameSession, OnProcessTerminate, and onHealthCheck.

    Call ProcessReady() with a designated port to notify the Amazon GameLift Servers service when ready to host game sessions.

```
// Copyright Amazon.com, Inc. or its affiliates. All Rights Reserved.
// SPDX-License-Identifier: Apache-2.0

#include "GameLiftUnrealAppGameMode.h"

#include "UObject/ConstructorHelpers.h"
#include "Kismet/GameplayStatics.h"

#if WITH_GAMELIFT
#include "GameLiftServerSDK.h"
#include "GameLiftServerSDKModels.h"
#endif

#include "GenericPlatform/GenericPlatformOutputDevices.h"

DEFINE_LOG_CATEGORY(GameServerLog);

AGameLiftUnrealAppGameMode::AGameLiftUnrealAppGameMode() :
    ProcessParameters(nullptr)
{
    // Set default pawn class to our Blueprinted character
    static ConstructorHelpers::FClassFinder<APawn> PlayerPawnBPClass(TEXT("/Game/ThirdPerson/Blueprints/BP_ThirdPersonCharacter"));

    if (PlayerPawnBPClass.Class != NULL)
    {
        DefaultPawnClass = PlayerPawnBPClass.Class;
    }

    UE_LOG(GameServerLog, Log, TEXT("Initializing AGameLiftUnrealAppGameMode..."));
}

void AGameLiftUnrealAppGameMode::BeginPlay()
{
    Super::BeginPlay();

#if WITH_GAMELIFT
    InitGameLift();
#endif
}

void AGameLiftUnrealAppGameMode::InitGameLift()
{
#if WITH_GAMELIFT
    UE_LOG(GameServerLog, Log, TEXT("Calling InitGameLift..."));

    // Getting the module first.
    FGameLiftServerSDKModule* GameLiftSdkModule = &FModuleManager::LoadModuleChecked<FGameLiftServerSDKModule>(FName("GameLiftServerSDK"));

    //Define the server parameters for a GameLift Anywhere fleet. These are not needed for a GameLift managed EC2 fleet.
    FServerParameters ServerParametersForAnywhere;

    bool bIsAnywhereActive = false;
    if (FParse::Param(FCommandLine::Get(), TEXT("glAnywhere")))
    {
        bIsAnywhereActive = true;
    }

    if (bIsAnywhereActive)
    {
        UE_LOG(GameServerLog, Log, TEXT("Configuring server parameters for Anywhere..."));

        // If GameLift Anywhere is enabled, parse command line arguments and pass them in the ServerParameters object.
        FString glAnywhereWebSocketUrl = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereWebSocketUrl="), glAnywhereWebSocketUrl))
        {
            ServerParametersForAnywhere.m_webSocketUrl = TCHAR_TO_UTF8(*glAnywhereWebSocketUrl);
        }

        FString glAnywhereFleetId = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereFleetId="), glAnywhereFleetId))
        {
            ServerParametersForAnywhere.m_fleetId = TCHAR_TO_UTF8(*glAnywhereFleetId);
        }

        FString glAnywhereProcessId = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereProcessId="), glAnywhereProcessId))
        {
            ServerParametersForAnywhere.m_processId = TCHAR_TO_UTF8(*glAnywhereProcessId);
        }
        else
        {
            // If no ProcessId is passed as a command line argument, generate a randomized unique string.
            FString TimeString = FString::FromInt(std::time(nullptr));
            FString ProcessId = "ProcessId_" + TimeString;
            ServerParametersForAnywhere.m_processId = TCHAR_TO_UTF8(*ProcessId);
        }

        FString glAnywhereHostId = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereHostId="), glAnywhereHostId))
        {
            ServerParametersForAnywhere.m_hostId = TCHAR_TO_UTF8(*glAnywhereHostId);
        }

        FString glAnywhereAuthToken = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereAuthToken="), glAnywhereAuthToken))
        {
            ServerParametersForAnywhere.m_authToken = TCHAR_TO_UTF8(*glAnywhereAuthToken);
        }

        FString glAnywhereAwsRegion = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereAwsRegion="), glAnywhereAwsRegion))
        {
            ServerParametersForAnywhere.m_awsRegion = TCHAR_TO_UTF8(*glAnywhereAwsRegion);
        }

        FString glAnywhereAccessKey = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereAccessKey="), glAnywhereAccessKey))
        {
            ServerParametersForAnywhere.m_accessKey = TCHAR_TO_UTF8(*glAnywhereAccessKey);
        }

        FString glAnywhereSecretKey = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereSecretKey="), glAnywhereSecretKey))
        {
            ServerParametersForAnywhere.m_secretKey = TCHAR_TO_UTF8(*glAnywhereSecretKey);
        }

        FString glAnywhereSessionToken = "";
        if (FParse::Value(FCommandLine::Get(), TEXT("glAnywhereSessionToken="), glAnywhereSessionToken))
        {
            ServerParametersForAnywhere.m_sessionToken = TCHAR_TO_UTF8(*glAnywhereSessionToken);
        }

        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_YELLOW);
        UE_LOG(GameServerLog, Log, TEXT(">>>> WebSocket URL: %s"), *ServerParametersForAnywhere.m_webSocketUrl);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Fleet ID: %s"), *ServerParametersForAnywhere.m_fleetId);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Process ID: %s"), *ServerParametersForAnywhere.m_processId);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Host ID (Compute Name): %s"), *ServerParametersForAnywhere.m_hostId);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Auth Token: %s"), *ServerParametersForAnywhere.m_authToken);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Aws Region: %s"), *ServerParametersForAnywhere.m_awsRegion);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Access Key: %s"), *ServerParametersForAnywhere.m_accessKey);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Secret Key: %s"), *ServerParametersForAnywhere.m_secretKey);
        UE_LOG(GameServerLog, Log, TEXT(">>>> Session Token: %s"), *ServerParametersForAnywhere.m_sessionToken);
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_NONE);
    }

    UE_LOG(GameServerLog, Log, TEXT("Initializing the GameLift Server..."));

    //InitSDK will establish a local connection with GameLift's agent to enable further communication.
    FGameLiftGenericOutcome InitSdkOutcome = GameLiftSdkModule->InitSDK(ServerParametersForAnywhere);
    if (InitSdkOutcome.IsSuccess())
    {
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_GREEN);
        UE_LOG(GameServerLog, Log, TEXT("GameLift InitSDK succeeded!"));
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_NONE);
    }
    else
    {
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_RED);
        UE_LOG(GameServerLog, Log, TEXT("ERROR: InitSDK failed : ("));
        FGameLiftError GameLiftError = InitSdkOutcome.GetError();
        UE_LOG(GameServerLog, Log, TEXT("ERROR: %s"), *GameLiftError.m_errorMessage);
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_NONE);
        return;
    }

    ProcessParameters = MakeShared<FProcessParameters>();

    //When a game session is created, Amazon GameLift Servers sends an activation request to the game server and passes along the game session object containing game properties and other settings.
    //Here is where a game server should take action based on the game session object.
    //Once the game server is ready to receive incoming player connections, it should invoke GameLiftServerAPI.ActivateGameSession()
    ProcessParameters->OnStartGameSession.BindLambda([=](Aws::GameLift::Server::Model::GameSession InGameSession)
        {
            FString GameSessionId = FString(InGameSession.GetGameSessionId());
            UE_LOG(GameServerLog, Log, TEXT("GameSession Initializing: %s"), *GameSessionId);
            GameLiftSdkModule->ActivateGameSession();
        });

    //OnProcessTerminate callback. Amazon GameLift Servers will invoke this callback before shutting down an instance hosting this game server.
    //It gives this game server a chance to save its state, communicate with services, etc., before being shut down.
    //In this case, we simply tell Amazon GameLift Servers we are indeed going to shutdown.
    ProcessParameters->OnTerminate.BindLambda([=]()
        {
            UE_LOG(GameServerLog, Log, TEXT("Game Server Process is terminating"));
            // First call ProcessEnding()
            FGameLiftGenericOutcome processEndingOutcome = GameLiftSdkModule->ProcessEnding();
            // Then call Destroy() to free the SDK from memory
            FGameLiftGenericOutcome destroyOutcome = GameLiftSdkModule->Destroy();
            // Exit the process with success or failure
            if (processEndingOutcome.IsSuccess() && destroyOutcome.IsSuccess()) {
                UE_LOG(GameServerLog, Log, TEXT("Server process ending successfully"));
            }
            else {
                if (!processEndingOutcome.IsSuccess()) {
                    const FGameLiftError& error = processEndingOutcome.GetError();
                    UE_LOG(GameServerLog, Error, TEXT("ProcessEnding() failed. Error: %s"),
                    error.m_errorMessage.IsEmpty() ? TEXT("Unknown error") : *error.m_errorMessage);
                }
                if (!destroyOutcome.IsSuccess()) {
                    const FGameLiftError& error = destroyOutcome.GetError();
                    UE_LOG(GameServerLog, Error, TEXT("Destroy() failed. Error: %s"),
                    error.m_errorMessage.IsEmpty() ? TEXT("Unknown error") : *error.m_errorMessage);
                }
            }
        });

    //This is the HealthCheck callback.
    //Amazon GameLift Servers will invoke this callback every 60 seconds or so.
    //Here, a game server might want to check the health of dependencies and such.
    //Simply return true if healthy, false otherwise.
    //The game server has 60 seconds to respond with its health status. Amazon GameLift Servers will default to 'false' if the game server doesn't respond in time.
    //In this case, we're always healthy!
    ProcessParameters->OnHealthCheck.BindLambda([]()
        {
            UE_LOG(GameServerLog, Log, TEXT("Performing Health Check"));
            return true;
        });

    //GameServer.exe -port=7777 LOG=server.mylog
    ProcessParameters->port = FURL::UrlConfig.DefaultPort;
    TArray<FString> CommandLineTokens;
    TArray<FString> CommandLineSwitches;

    FCommandLine::Parse(FCommandLine::Get(), CommandLineTokens, CommandLineSwitches);

    for (FString SwitchStr : CommandLineSwitches)
    {
        FString Key;
        FString Value;

        if (SwitchStr.Split("=", &Key, &Value))
        {
            if (Key.Equals("port"))
            {
                ProcessParameters->port = FCString::Atoi(*Value);
            }
        }
    }

    //Here, the game server tells Amazon GameLift Servers where to find game session log files.
    //At the end of a game session, Amazon GameLift Servers uploads everything in the specified 
    //location and stores it in the cloud for access later.
    TArray<FString> Logfiles;
    Logfiles.Add(TEXT("GameLiftUnrealApp/Saved/Logs/server.log"));
    ProcessParameters->logParameters = Logfiles;

    //The game server calls ProcessReady() to tell Amazon GameLift Servers it's ready to host game sessions.
    UE_LOG(GameServerLog, Log, TEXT("Calling Process Ready..."));
    FGameLiftGenericOutcome ProcessReadyOutcome = GameLiftSdkModule->ProcessReady(*ProcessParameters);

    if (ProcessReadyOutcome.IsSuccess())
    {
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_GREEN);
        UE_LOG(GameServerLog, Log, TEXT("Process Ready!"));
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_NONE);
    }
    else
    {
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_RED);
        UE_LOG(GameServerLog, Log, TEXT("ERROR: Process Ready Failed!"));
        FGameLiftError ProcessReadyError = ProcessReadyOutcome.GetError();
        UE_LOG(GameServerLog, Log, TEXT("ERROR: %s"), *ProcessReadyError.m_errorMessage);
        UE_LOG(GameServerLog, SetColor, TEXT("%s"), COLOR_NONE);
    }

    UE_LOG(GameServerLog, Log, TEXT("InitGameLift completed!"));
#endif
} 
```

# Step 6: ntegrate your client game map

[Integrate your client game map](https://docs.aws.amazon.com/gameliftservers/latest/developerguide/unreal-plugin-integrate.html#unreal-plugin-anywhere-integrate-simple-client)

The startup game map contains blueprint logic and UI elements that already include basic code to request game sessions and use connection information to connect to a game session. You can use the map as is or modify these as needed. Use the startup game map with other game assets, such as the Third Person template project provided by Unreal Engine. These assets are available in Content Browser. You can use them to test the plugin's deployment workflows, or as a guide to create a custom backend service for your game.

The startup map has the following characteristics:

    It includes logic for both an Anywhere fleet and a managed EC2 fleet. When you run your client, you can choose to connect to either fleet.

    Client functionality includes find a game session (SearchGameSessions()), create a new game session (CreateGameSession()), and join a game session directly.

    It gets a unique player ID from your project's Amazon Cognito user pool (this is part of a deployed Anywhere solution). 


<b>To use the startup game map</b>

1. In the UE editor, open the Project Settings, Maps & Modes page, and expand the Default Maps section.

2. For Editor Startup Map, select "StartupMap" from the dropdown list. You might need to search for the file, which is located in ... > Unreal Projects/[project-name]/Plugins/Amazon GameLift Servers Plugin Content/Maps.

3. For Game Default Map, select the same "StartupMap" from the dropdown list.

4. For Server Default Map, select "Lv1_ThirdPerson" for Unreal Engine 5.6 or later, or "ThirdPersonMap" for earlier versions. This is a default map included in your game project. This map is designed for two players in the game.

5. Open the details panel for the server default map. Set GameMode Override to "None".

6. Expand the Default Modes section, and set Global Default Server Game Mode to the game mode you updated for your server integration. 

After you've made these changes to your project, you're ready to build your game components.

For Unreal Engine 5.6 or later, if you cannot move the character after connecting to the game server, update the BP_ThirdPersonCharacter blueprint to add input mapping context for IMC_Default and IMC_MouseLook as shown below:

![](./AWS%20Gamelift/Input%20Mapping%20Conext%20Issue.png)

# Step 7: Building the Dedicated Server and Client App

Standard Unreal Engine installs from the Launcher cannot do this; you must use your Source-built Engine (the one you compiled from GitHub).

Before starting, ensure your Source-built Engine is open and your project is loaded.

1. Open your game project in a source-built version of the Unreal Engine editor.

2. If using Unreal Engine 5.6 or later, go to Edit, Project Settings, Packaging. Find Cook everything in the project content directory and enable it.

3. Use the editor to package your game client and server builds. 

    - Choose a target. Go to Platforms, Windows and select one of the following:
        - Server: [your-application-name]Server
        - Client: [your-application-name]Client
    - Start the build. Go to Platform, Windows, Package Project.

Each packaging process generates an executable: [your-application-name]Client.exe or [your-application-name]Server.exe.

In the plugin, set the paths to the client and server build executables on your local workstation.

# Set Up Anywhere Fleet for LocalTesting

[Plugin for Unreal: Host your game locally with Amazon GameLift Servers Anywhere](https://docs.aws.amazon.com/gameliftservers/latest/developerguide/unreal-plugin-anywhere.html)

1. Open the GameLift PLugin and select "Test locally with Anywhere" in the "Choose a hosting option"

    ![](./AWS%20Gamelift/Anywhere.png)

2. Set the path to Server exe 

    ![](./AWS%20Gamelift/ServerExecutatbleAnywhere.png)

3. Create an Anywhere Fleet

    ![](./AWS%20Gamelift/Create%20Anywhere%20Fleet.png)

4. Register your computer as the server

5. Click on "Start Server"

6. Click on "Start Client"


# Setup EC2 Fleet

[Plugin for Unreal: Deploy your game to a managed EC2 fleet](https://docs.aws.amazon.com/gameliftservers/latest/developerguide/unreal-plugin-ec2.html)



# Notable Errors

1. You might run into this error

```
Assertion failed: CurrentApplication.IsValid() [File:D:\Github Repos\Unreal Engine 5.7\UnrealEngine\Engine\Source\Runtime\Slate\Public\Framework\Application\SlateApplication.h] [Line: 321] 
```

The error you are seeing is a classic "Client Logic on a Dedicated Server" crash.

Specifically, your GameLift server is crashing because it is trying to access Slate (Unreal’s UI system) to check for a virtual joystick. Dedicated Servers are "headless," meaning they have no GPU, no window, and—crucially—no FSlateApplication instance. When the code calls FSlateApplication::Get(), it fails the assertion because the application doesn't exist.

In the PlayerController.cpp, find BeginPlay() and add a check to see if it is a dedicated server first, then execute the joystick inputs.

```
if (!IsNetMode(NM_DedicatedServer) && ShouldUseTouchControls() && IsLocalPlayerController())
{
	// spawn the mobile controls widget
	MobileControlsWidget = CreateWidget<UUserWidget>(this, MobileControlsWidgetClass);

	if (MobileControlsWidget)
	{
		// add the controls to the player screen
		MobileControlsWidget->AddToPlayerScreen(0);

	} else {

		UE_LOG(LogMMO_AWS, Error, TEXT("Could not spawn mobile controls widget."));

	}

}
```