# Overview

[Gemini Conversation](https://gemini.google.com/app/d6cd4c4ae349b4ad)



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

