
---
title: Peer-to-peer or Channel Messaging
description: v1.0
platform: iOS
updatedAt: Fri Oct 16 2020 06:30:23 GMT+0800 (CST)
---
# Peer-to-peer or Channel Messaging

You can use this guide to quickly start messaging with the [Agora RTM iOS SDK](https://docs.agora.io/en/Real-time-Messaging/downloads). 




## Prerequisites



- Xcode 9.0 or later
- An iOS device running on iOS 8.0 or later.

- A valid Agora account. ([Sign up](https://sso.agora.io/en/signup) for free)

<div class="alert note">Open the specified ports in <a href="https://docs.agora.io/en/Agora%20Platform/firewall?platform=All%20Platforms">Firewall Requirements</a > if your intranet has a firewall.</div> 



## Set up the development environment

This section describes how to get an App ID, create an iOS project, and integrate the [Agora RTM SDK for iOS](https://docs.agora.io/en/Real-time-Messaging/downloads) into your project. 

### <a name="appid"></a>Get an App ID

You can skip to [Create an iOS project](#create) if you already have an App ID. 

<details>
	<summary><font color="#3ab7f8">Get an App ID</font></summary>
	
1. Sign up for a developer account at [Agora Console](https://dashboard.agora.io/). See [Sign in and Sign up](../../en/Real-time-Messaging/sign_in_and_sign_up.md).

2. Click ![](https://web-cdn.agora.io/docs-files/1551254998344) in the left navigation menu to enter the [**Project Management**](https://dashboard.agora.io/projects) page.

3. Click **Create**. 

![](https://web-cdn.agora.io/docs-files/1574924327108)

4.  Enter your project name and select your authentication mechanism ("App ID") in the dialog box.

![](https://web-cdn.agora.io/docs-files/1574924446798)
	
5. Click **Submit** and you can find the **App ID** of your newly created project.

![](https://web-cdn.agora.io/docs-files/1574924570426)
</details>
	
### <a name="create"></a> Create an iOS project

Now, let's build a project from scratch. Skip to [Integrate the SDK](#Integrate) if you have already created a project.

<details>
	<summary><font color="#3ab7f8">Create an iOS project</font></summary>

1. Open **Xcode** and click **Create a new Xcode project**.
2. Choose **Single View App** as the template and click **Next**.
3. Input the project information, such as the project name, team, organization name, and language, and click **Next**.
	
  **Note**: If you haven't added any team information, you will see an **Add account...** button. Click it, input your Apple ID, and click **Next** to add your team.
4. Choose the storage path of the project and click **Create**.
5. Connect your iOS device to your computer.
6. Go to the **TARGETS > Project Name > General > Signing** menu, choose **Automatically manage signing**, and then click **Enable Automatic** on the pop-up window.
	
	![](https://web-cdn.agora.io/docs-files/1568803558097)
</details>

### <a name="Integrate"></a>Integrate the SDK


Choose either of the following methods to integrate the Agora SDK into your project.

**Method 1: Automatically integrate the SDK with CocoaPods**

1. Ensure that you have installed **CocoaPods** before the following steps. See the installation guide in [Getting Started with CocoaPods](https://guides.cocoapods.org/using/getting-started.html#getting-started).
2. In **Terminal**, go to the project path and run the `pod init` command to create a **Podfile** in the project folder.
3. Open the **Podfile**, delete all contents and input the following contents. Remember to change **Your App** to the target name of your project.
```
platform :iOS, '9.0' use_frameworks!
target 'Your App' do
    pod 'AgoraRtm_iOS'
end
```
4. Go back to **Terminal**, and run the `pod update` command to update the local libraries.
5. Run the `pod install` command to install the Agora SDK. Once you successfully install the SDK, it shows `Pod installation complete!` in Terminal, and you can see an **xcworkspace** file in the project folder.
6. Open the generated **xcworkspace** file in **Xcode**.

**Method 2: Manually add the SDK files**

1. Download the latest version of the [Agora RTM iOS SDK](https://docs.agora.io/cn/Agora%20Platform/downloads), and extract the files from.
2. Copy the **AgoraRtmKit.framework** file in the **libs** folder to the project folder.
3. In **Xcode**, go to the **TARGETS > Project Name > Build Phases > Link Binary with Libraries** menu, and click <b>+</b> to add the following frameworks and libraries. To add the **AgoraRtmKit.framework** file, remember to click **Add Other...** after clicking **+**.

	- AgoraRtmKit.framework
	- libc++.tbd
	- libresolv.tbd
	- SystemConfiguration.framework
	- CoreTelephony.framework
	- CoreWLAN.framework (macOS only)


## Implement peer-to-peer and channel messaging

This section provides API call sequence diagrams, sample codes, and considerations related to peer-to-peer messaging and channel messaging.


### <a name = "create"></a>Create and Initialize an AgoraRtmKit Instance

Call the [initWithAppId](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/initWithAppId:delegate:) method to create an [AgoraRtmKit](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html) instance. You need to: 

   - Pass the App ID issued by Agora to you. Only apps with the same App ID can join the same channel.
   - Specify an event handler. The Agora RTM SDK process event handlers with the [AgoraRtmDelegate](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmDelegate.html) instance to inform the app about runtime events, such as connection state changes and receiving peer-to-peer messages.

```objective-c
@interface ViewController ()<AgoraRtmChannelDelegate, AgoraRtmDelegate>

@property(nonatomic, strong)AgoraRtmKit* kit;

@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    
    // Create an AgoraRtmKit instance.
    _kit = [[AgoraRtmKit alloc] initWithAppId:YOUR_APP_ID delegate:self];
	// Note: The SDK does not trigger the callback if the AgoraRtmKit reference is not held and released.
	// A token is mandatory if security options are enabled for your appId.
}
```

### <a name = "login"></a>Log in Agora's RTM System


Call the [loginByToken](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/loginByToken:user:completion:) method to log in Agora's RTM system. You need to: 

- Pass a token that identifies the role and permission of the user. Set `token` as `"nil"` for low-security requirements. A token is generated at the server of the app. For more information, see [Token Security](../../en/Real-time-Messaging/rtm_token.md).
- Pass a user ID that identifies the user. The `userId` parameter must not include non-printable characters, and can neither exceed 64 bytes in length nor start with a space. Do not set `userId` as `"nil"`.
- Pass the [AgoraRtmLoginBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmLoginBlock.html) and [connectionStateChanged](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmDelegate.html#//api/name/rtmKit:connectionStateChanged:reason:) callbacks, which indicate whether the login succeeds or fails.

```objective-c
[_kit loginByToken:nil user:@"testuser" completion:^(AgoraRtmLoginErrorCode errorCode) {
    if (errorCode != AgoraRtmLoginErrorOk) {
        NSLog(@"login failed %@", @(errorCode));
    } else {
        NSLog(@"login success");
    }
}];

#pragma AgoraRtmDelegate
...
- (void)rtmKit:(AgoraRtmKit *)kit connectionStateChanged:(AgoraRtmConnectionState)state reason:(AgoraRtmConnectionChangeReason)reason
{
    NSLog(@"connection state changed to %@", @(reason));
}
```

You can call the [logoutWithCompletion](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/logoutWithCompletion:) method to log out of Agora's RTM system.


```objective-c
[_kit logoutWithCompletion:^(AgoraRtmLogoutErrorCode errorCode) {
    if (errorCode != AgoraRtmLogoutErrorOk) {
        NSLog(@"login failed %@", @(errorCode));
    } else {
        NSLog(@"login success");
    }
}];
```

After calling the [logoutWithCompletion](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/logoutWithCompletion:) method to log out of Agora's RTM system, you can call the [loginByToken](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/loginByToken:user:completion:)  method again to switch to another account.


### <a name = "sendpeer"></a>Peer-to-peer messaging

After logging in Agora's RTM system, you can send and receive a peer-to-peer message.

#### Send a peer-to-peer message

Call the [sendMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/sendMessage:toPeer:completion:) method to send a peer-to-peer message to a specific user (peer).

- Pass the user ID (`peerId`) of the receiver.
- Pass the [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) instance. You can call the [initWithText](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html#//api/name/initWithText:) method in the [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) interface to create an [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) instance and create the text message.
- Pass the `message` to be sent.
- Pass the [AgoraRtmSendPeerMessageBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmSendPeerMessageBlock.html) and [messageReceived](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmDelegate.html#//api/name/rtmKit:messageReceived:fromPeer:) callbacks, which return the state of sending the message, such as whether or not the peer-to-peer message is received by the peer.

```objective-c
- (void)sendYourPeerToPeerMessage {
    [_kit sendMessage:[[AgoraRtmMessage alloc] initWithText:@"testmsg"] toPeer:@"peer" completion:^(AgoraRtmSendPeerMessageErrorCode errorCode) {
        if (errorCode == AgoraRtmSendPeerMessageErrorOk) {
            NSLog(@"sent success");
        }
    }];
}

#pragma AgoraRtmDelegate
...
- (void)rtmKit:(AgoraRtmKit *)kit messageReceived:(AgoraRtmMessage *)message fromPeer:(NSString *)peerId
{
    NSLog(@"message received from %@: %@", message.text, peerId);
}
```
#### Receive a peer-to-peer message

- The sender receives the [AgoraRtmSendPeerMessageBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmSendPeerMessageBlock.html) callback with the state of sending the peer-to-peer message.
- The receiver receives the [messageReceived](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmDelegate.html#//api/name/rtmKit:messageReceived:fromPeer:) callback with the sent message and the user ID (`peerId`) of the sender.

> You cannot reuse an Agora [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) object that has been received by a user.


### <a name = "sendchannel"></a>Channel messaging

After logging in Agora's RTM system, you can send a channel message.

#### Create an AgoraRtmChannel instance and join a channel

1. Call the [createChannelWithId](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/createChannelWithId:delegate:) method in the [AgoraRtmChannel](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html) instance to create an Agora RTM channel.  You need to: 

 - Pass the channel ID. The `channelId` parameter must be in the string format, less than 64 bytes, and cannot be empty or set as `"nil"`.
 - Specify a reference to [AgoraRtmChannelDelegate](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmChannelDelegate.html), through which the Agora RTM SDK informs the app about channel events, such as receiving channel messages and a user joining or leaving a channel.

2. Call the [joinWithCompletion](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html#//api/name/joinWithCompletion:) method in the [AgoraRtmChannel](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html) instance to join a channel. The [AgoraRtmJoinChannelBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmJoinChannelBlock.html) and [memberJoined](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmChannelDelegate.html#//api/name/channel:memberJoined:) callbacks indicate whether this method call succeeds or fails.

```objective-c
@interface ViewController ()<AgoraRtmChannelDelegate, AgoraRtmDelegate>

...
@property(nonatomic, strong)AgoraRtmChannel* channel;

@implementation
- (void)createAndJoinChannel {
    _channel = [_kit createChannelWithId:@"testchannel" delegate:self];
    [_channel joinWithCompletion:^(AgoraRtmJoinChannelErrorCode state) {
        if(state == AgoraRtmJoinChannelErrorOk) {
            NSLog(@"join success");
        } else {
            NSLog(@"join failed: %@", @(state));
        }
    }];
}
#pragma AgoraRtmChannelDelegate
...
- (void)channel:(AgoraRtmChannel *)channel memberLeft:(AgoraRtmMember *)member
{
    NSLog(@"%@ left channel %@", member.userId, member.channelId);
}

- (void)channel:(AgoraRtmChannel *)channel memberJoined:(AgoraRtmMember *)member
{
    NSLog(@"%@ joined channel %@", member.userId, member.channelId);
}
```

#### Send a channel message

Call the [sendMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html#//api/name/sendMessage:completion:) method in the [AgoraRtmChannel](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html) instance to send a channel message. You need to: 

- Pass the [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) instance. You can call the `initWithText` method in the [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) interface to create an [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) instance and create the text message.
- Pass the [AgoraRtmSendChannelMessageBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmSendChannelMessageBlock.html) and [messageReceived](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmChannelDelegate.html#//api/name/channel:messageReceived:fromMember:) callbacks, which return the state of sending the message, such as whether or not the channel message is received by the server.

```objective-c
- (void)sendYourChannelMessage {
    [_channel sendMessage:[[AgoraRtmMessage alloc] initWithText:@"channelmsg"] completion:^(AgoraRtmSendChannelMessageErrorCode errorCode) {
        if(errorCode == AgoraRtmSendChannelMessageErrorOk) {
            NSLog(@"sent success");
        }
    }];
}

#pragma AgoraRtmDelegate
...
- (void)channel:(AgoraRtmChannel *)channel messageReceived:(AgoraRtmMessage *)message fromMember:(AgoraRtmMember *)member
{
    NSLog(@"message received from %@ in channel %@: %@", message.text, member.channelId, member.userId);
}
```

#### Receive a channel message

- The sender receives the [AgoraRtmSendChannelMessageBlock](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Blocks/AgoraRtmSendChannelMessageBlock.html) callback with the state of sending the channel message.
- The receivers receive the [messageReceived](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Protocols/AgoraRtmChannelDelegate.html#//api/name/channel:messageReceived:fromMember:) callback with the sent message and the user ID (`member`) of the sender.

#### Leave a channel
Call the [leaveWithCompletion](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html#//api/name/leaveWithCompletion:) method in the [AgoraRtmChannel](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmChannel.html) instance to leave a channel. 

```objective-c
[_channel leaveWithCompletion:^(AgoraRtmLeaveChannelErrorCode state) {
    if(state == AgoraRtmLeaveChannelErrorOk) {
        NSLog(@"leave success");
    } else {
        NSLog(@"leave failed: %@", @(state));
    }
}];
```


## Considerations


- The Agora RTM SDK supports creating multiple [AgoraRtmKit](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html) instances that are independent of each other. 

-  To send and receive peer-to-peer or channel messages, ensure that you have successfully logged in the Agora RTM system (i.e., ensure that you have received `AgoraRtmLoginErrorOk`). 

- To use any of the channel features, you must first call the [createChannelWithId](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/createChannelWithId:delegate:) method to create a channel instance. 
- You can create multiple channel instances for each [AgoraRtmKit](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html) instance, but you can only join a maximum of 20 channels at the same time. The `channelId` parameter needs to be channel-specific.

- When you leave a channel and do not want to join it again, you can call the [destroyChannelWithId](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmKit.html#//api/name/destroyChannelWithId:) method to release all resources used by the channel instance.

- You cannot reuse a received [AgoraRtmMessage](https://docs.agora.io/en/Real-time-Messaging/API%20Reference/RTM_oc/Classes/AgoraRtmMessage.html) instance.

