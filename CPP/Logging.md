# Overview

Logging in UE


# UE_Log

## Using TEXT() with Variables


```
FString FleetIdStr = ...;
FString CredentialsNameStr = ...;
FString CustomLocationStr = ...;

UE_LOG(LogTemp, Warning, TEXT("FleetIdStr %s CredentialsNameStr %s CustomLocationStr %s"), 
    *FleetIdStr, *CredentialsNameStr, *CustomLocationStr)
```