# Overview

Using the AWS C++ SDK for Cognito


# User Signup

- Imports

    ```
    #include <aws/cognito-idp/CognitoIdentityProviderClient.h>
    #include <aws/cognito-idp/model/SignUpRequest.h>
    ```

- Callbacks

    Header
    ```
    DECLARE_DYNAMIC_MULTICAST_DELEGATE_OneParam(FOnSignUpComplete, bool, bSuccess);


    UCLASS()
    class MMO_AWS_API UTestSubsystem : public UGameInstanceSubsystem
    {
    public:
        UPROPERTY(BlueprintAssignable)
        FOnSignUpComplete OnSignUpComplete;
    }

    ```

    Binding events to the Delegate

    ```
    OnSignUpComplete.BindLambda([](bool bSuccess) {
        UE_LOG(LogTemp, Warning, TEXT("Task finished!"));
    });
    ```

    Triggering the Delegate

    ```
    OnSignUpComplete.Broadcast(true);
    ```




- Sign Up Logic

    ```
    void UMMOOnlineSubsystem::SignUp(FString Username, FString Password)
    {
        Aws::Client::ClientConfiguration clientConfig;
        clientConfig.region = "us-east-1";
        // 2. Client (Use a shared pointer)
        auto CognitoClient = std::make_shared<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient>(clientConfig);

        const UAWSConfig* Settings = GetDefault<UAWSConfig>();
        FString CognitoClientId = Settings->CognitoClientId;


        // 3. Request setup
        Aws::CognitoIdentityProvider::Model::SignUpRequest Request;
        Request.SetClientId(TCHAR_TO_UTF8(*CognitoClientId));
        Request.SetUsername(TCHAR_TO_UTF8(*Username));
        Request.SetPassword(TCHAR_TO_UTF8(*Password));

        UE_LOG(LogTemp, Warning, TEXT("Attempting Sign-Up - Username: %s, Password: %s"), *Username, *Password);

        Aws::CognitoIdentityProvider::Model::AttributeType EmailAttribute;
        EmailAttribute.SetName("email");
        EmailAttribute.SetValue(TCHAR_TO_UTF8(*Username));
        Request.AddUserAttributes(EmailAttribute);

        // Capture 'this' as a WeakPtr
        TWeakObjectPtr<UMMOOnlineSubsystem> WeakSelf(this);

        // 4. Async Call
        // We capture 'this' and the 'CognitoClient' to ensure the client stays alive 
        // until the callback finishes.
        CognitoClient->SignUpAsync(Request,
            [WeakSelf, CognitoClient](const Aws::CognitoIdentityProvider::CognitoIdentityProviderClient*,
                const Aws::CognitoIdentityProvider::Model::SignUpRequest&,
                const Aws::CognitoIdentityProvider::Model::SignUpOutcome& Outcome,
                const std::shared_ptr<const Aws::Client::AsyncCallerContext>&)
            {
                // Back to Game Thread for UI updates
                AsyncTask(ENamedThreads::GameThread, [this, Outcome]() {
                    if (Outcome.IsSuccess())
                    {
                        OnSignUpComplete.Broadcast(true);
                    }
                    else
                    {
                        // Log the Exception Name for better debugging
                        FString ErrorType = UTF8_TO_TCHAR(Outcome.GetError().GetExceptionName().c_str());
                        FString ErrorMsg = UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str());
                        UE_LOG(LogTemp, Error, TEXT("Sign-up failed (%s): %s"), *ErrorType, *ErrorMsg);
                        OnSignUpComplete.Broadcast(false);
                    }
                    });
            });
    }
    ```

# User Signin

- AWS COnsole Config

    1. Make sure Authentication flows has "Sign in with username and password: ALLOW_USER_PASSWORD_AUTH" enabled when you go to Cognito > User Pools > App clients > Edit > check "Sign in with username and password: ALLOW_USER_PASSWORD_AUTH"

    ![](./AWS%20Cognito/App%20Client.png)

    ![](./AWS%20Cognito/App%20Client%20Password%20Auth%20Flow%20Enabled.png)



- Imports

    ```
    #include "aws/cognito-idp/model/InitiateAuthRequest.h"
    ```

- Sign in

    ```
    void UCognitoSubsystem::SignIn(FString Username, FString Password)
    {
        const UAWSConfiguration* Settings = GetDefault<UAWSConfiguration>();

        // Safety check for Settings
        if (!Settings || !CognitoClient) return;

        Aws::CognitoIdentityProvider::Model::InitiateAuthRequest Request;
        // Explicitly convert to std::string to ensure the Request owns the data
        Request.SetClientId(TCHAR_TO_UTF8(*Settings->CognitoClientId));
        Request.SetAuthFlow(Aws::CognitoIdentityProvider::Model::AuthFlowType::USER_PASSWORD_AUTH);

        Request.AddAuthParameters("USERNAME", TCHAR_TO_UTF8(*Username));
        Request.AddAuthParameters("PASSWORD", TCHAR_TO_UTF8(*Password));

        // Capture 'this' as a WeakPtr
        TWeakObjectPtr<UCognitoSubsystem> WeakSelf(this);

        CognitoClient->InitiateAuthAsync(Request,
            [WeakSelf](const auto* Client, const auto& Req, const auto& Outcome, const auto& Context)
            {
                AsyncTask(ENamedThreads::GameThread, [WeakSelf, Outcome]()
                {
                    // Check if 'this' is still valid before doing anything!
                    if (!WeakSelf.IsValid())
                    {
                        return;
                    }

                    if (Outcome.IsSuccess())
                    {
                        // Extract the tokens
                        const auto& Result = Outcome.GetResult().GetAuthenticationResult();

                        FString AccessToken = UTF8_TO_TCHAR(Result.GetAccessToken().c_str());
                        FString IdToken = UTF8_TO_TCHAR(Result.GetIdToken().c_str());
                        FString RefreshToken = UTF8_TO_TCHAR(Result.GetRefreshToken().c_str());
                        int32 ExpiresIn = Result.GetExpiresIn();

                        // Store these in your class or pass them to your Delegate
                        // WeakSelf->MyAccessToken = AccessToken;
                        // WeakSelf->MyRefreshToken = RefreshToken;

                        UE_LOG(LogTemp, Log, TEXT("Sign-in Success! Access Token length: %d"), AccessToken.Len());
                        //WeakSelf->OnSignInComplete.Broadcast(true);
                    }
                    else
                    {
                        FString Error = UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str());
                        UE_LOG(LogTemp, Error, TEXT("Sign-in Failed: %s"), *Error);
                        //WeakSelf->OnSignInComplete.Broadcast(false);
                    }
                });
            });
    }
    ```

# Get User Info

- Imports

    ```
    #include "aws/cognito-idp/model/GetUserRequest.h"
    ```

- Get User

    ```
    using namespace Aws::CognitoIdentityProvider::Model;

    GetUserRequest Request;
    Request.SetAccessToken(TCHAR_TO_UTF8(*AccessToken));

    CognitoClient->GetUserAsync(Request, 
        [this](const auto* Client, const auto& Req, const auto& Outcome, const auto& Context)
        {
            AsyncTask(ENamedThreads::GameThread, [this, Outcome]()
            {
                if (Outcome.IsSuccess())
                {
                    // The UserAttributes list contains things like "email_verified"
                    auto Attributes = Outcome.GetResult().GetUserAttributes();
                    bool bIsEmailVerified = false;

                    for (const auto& Attr : Attributes)
                    {
                        if (Attr.GetName() == "email_verified" && Attr.GetValue() == "true")
                        {
                            bIsEmailVerified = true;
                            break;
                        }
                    }

                    UE_LOG(LogTemp, Log, TEXT("User verified status: %s"), bIsEmailVerified ? TEXT("True") : TEXT("False"));
                    OnVerificationChecked.Broadcast(bIsEmailVerified);
                }
                else
                {
                    UE_LOG(LogTemp, Error, TEXT("Failed to get user data: %s"), 
                        UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str()));
                }
            });
        });
    ```

# Sign Out (Global)

- Import

    ```
    #include "aws/cognito-idp/model/GlobalSignOutRequest.h"

    ```

- Sign Out Code

    ```
    void UMMOOnlineSubsystem::SignOut()
    {

        if (!*AccessToken) {
            UE_LOG(LogTemp, Log, TEXT("No access token."));

            return;
        }
        using namespace Aws::CognitoIdentityProvider;
        using namespace Aws::CognitoIdentityProvider::Model;

        // 1. Initialize Client (Usually persistent in your Auth Manager)
        Aws::Client::ClientConfiguration clientConfig;
        clientConfig.region = "us-east-1";
        auto CognitoClient = std::make_shared<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient>(clientConfig);

        // 2. Build Request
        GlobalSignOutRequest Request;
        Request.SetAccessToken(TCHAR_TO_UTF8(*AccessToken));

        // 3. Call Async (Recommended for Unreal to prevent UI hitching)
        CognitoClient->GlobalSignOutAsync(Request,
            [this, CognitoClient](const CognitoIdentityProviderClient*, const GlobalSignOutRequest&, const GlobalSignOutOutcome& Outcome, const std::shared_ptr<const Aws::Client::AsyncCallerContext>&)
            {
                if (Outcome.IsSuccess())
                {
                    UE_LOG(LogTemp, Log, TEXT("Successfully signed out globally."));
                    // Clear local variables/tokens here
                }
                else
                {
                    UE_LOG(LogTemp, Error, TEXT("Sign out failed: %s"),
                        UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str()));
                }
            });
    }
    ```

# Resend verification code

- Imports

    ```
    #include "aws/cognito-idp/model/ResendConfirmationCodeRequest.h"
    #include <aws/cognito-idp/CognitoIdentityProviderClient.h>
    ```
- Header

    ```


    UCLASS()
    class MMO_AWS_API UTestSubsystem : public UGameInstanceSubsystem
    {

    public:
        TSharedPtr<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient> CognitoClient;
        Aws::Client::ClientConfiguration ClientConfig;

    }
    
    ```
    
- Send Code

    ```
    using namespace Aws::CognitoIdentityProvider::Model;

    FString MyClientId = TEXT("...");
    FString Username = TEXT("...");

    ResendConfirmationCodeRequest Request;
    Request.SetClientId(TCHAR_TO_UTF8(*MyClientId));
    Request.SetUsername(TCHAR_TO_UTF8(*Username));

    if (!CognitoClient.IsValid())
    {
        ClientConfig.region = "us-east-1";
        CognitoClient = MakeShared<Aws::CognitoIdentityProvider::CognitoIdentityProviderClient>(ClientConfig);
    }

    CognitoClient->ResendConfirmationCodeAsync(Request, 
        [this](const auto* Client, const auto& Req, const auto& Outcome, const auto& Context)
        {
            AsyncTask(ENamedThreads::GameThread, [this, Outcome]() {
                if (Outcome.IsSuccess()) {
                    UE_LOG(LogTemp, Log, TEXT("Confirmation code resent!"));
                } else {
                    UE_LOG(LogTemp, Error, TEXT("Resend failed: %s"), 
                        UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str()));
                }
            });
        });
    ```

# Confirm User

- Imports

    ```
    #include "aws/cognito-idp/CognitoIdentityProviderClient.h"
    #include "aws/cognito-idp/model/ConfirmSignUpRequest.h"
    ```

- Confirmation Code

    ```
    bool UMMOOnlineSubsystem::ConfirmUser(FString Username, FString ConfirmationCode)
    {
        Aws::Client::ClientConfiguration clientConfig;
        // Optional: clientConfig.region = "us-east-1";

        Aws::CognitoIdentityProvider::CognitoIdentityProviderClient client(clientConfig);

        const UAWSConfig* Settings = GetDefault<UAWSConfig>();

        if (!Settings) return false;

        Aws::CognitoIdentityProvider::Model::ConfirmSignUpRequest request;
        request.SetClientId(TCHAR_TO_UTF8(*Settings->CognitoClientId));
        request.SetUsername(TCHAR_TO_UTF8(*Username));
        request.SetConfirmationCode(TCHAR_TO_UTF8(*ConfirmationCode));

        auto Outcome = client.ConfirmSignUp(request);

        if (Outcome.IsSuccess()) {
            UE_LOG(LogTemp, Log, TEXT("Cognito: User Confirmed Successfully!"));
            return true;
        }
        else {
            FString ErrorMsg = UTF8_TO_TCHAR(Outcome.GetError().GetMessage().c_str());
            UE_LOG(LogTemp, Error, TEXT("Cognito Error: %s"), *ErrorMsg);
            return false;
        }


        return false;
    }
    ```