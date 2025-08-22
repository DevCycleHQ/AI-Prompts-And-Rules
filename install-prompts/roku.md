# DevCycle Roku SDK Installation Prompt

You are helping to install and configure the DevCycle Roku SDK in a Roku channel application. Follow this complete guide to successfully integrate DevCycle feature flags. Do not install any Variables as part of this process, the user can ask for you to do that later.

**Do not use the SDK for:**

- Web applications (use JavaScript/React SDKs instead)
- Mobile applications (use iOS/Android/React Native/Flutter SDKs instead)
- Smart TV platforms other than Roku (use appropriate platform SDKs)

If you detect that the user is trying to have you install the Roku SDK in an application where it will not work, please stop what you are doing and advise the user which SDK they should be using.

## Required Information

Before proceeding, use your own analysis, the DevCycle MCP or web search to ensure you have:

- [ ] A DevCycle account and project set up
- [ ] A Development environment **Mobile SDK Key** (starts with `dvc_mobile_`)
- [ ] Roku development environment set up
- [ ] BrightScript project for Roku channel
- [ ] The most recent DevCycle Roku SDK version to install

**Security Note:** Use a MOBILE SDK key for Roku apps. Store it securely in your channel configuration.

## SDK Key Configuration

**IMPORTANT:** After obtaining the SDK key, you must set it up properly:

1. **First, attempt to add the SDK key to your channel configuration or manifest**:

   - Consider using a config file that's not committed to version control
   - Or set it up in your build process to inject the key during packaging

2. **If you cannot create or modify configuration files** (due to system restrictions or security policies), ask the user:

   - "I'm unable to create/modify configuration files. Would you like me to:
     a) Temporarily hardcode the SDK key for testing purposes (you'll need to update it later for production)
     b) Provide you with the SDK key and instructions so you can set it up yourself?"

3. **Based on the user's response:**
   - If they choose hardcoding: Add a clear comment indicating this is temporary and should be replaced with secure configuration
   - If they choose manual setup: Provide them with the SDK key and clear instructions on how to set it up securely

**Note:** Always prefer configuration files or build-time injection over hardcoding for security reasons.

## Installation Steps

### 1. Download the DevCycle Roku SDK

Download the latest DevCycle Roku SDK from the GitHub releases or package repository and add it to your project.

### 2. Add DevCycle SDK to Your Project

Copy the DevCycle SDK files to your Roku project's source directory:

```text
your-roku-project/
├── source/
│   ├── devcycle/
│   │   └── (DevCycle SDK files)
│   └── main.brs
```

### 3. Initialize DevCycle in Your Channel

In your main BrightScript file (typically `source/main.brs` or `source/Main.brs`):

```brightscript
sub Main()
    ' Initialize screen and message port
    screen = CreateObject("roSGScreen")
    m.port = CreateObject("roMessagePort")
    screen.setMessagePort(m.port)

    ' Initialize DevCycle
    initializeDevCycle()

    ' Create and show your scene
    scene = screen.CreateScene("MainScene")
    screen.show()

    ' Main event loop
    while(true)
        msg = wait(0, m.port)
        msgType = type(msg)

        if msgType = "roSGScreenEvent"
            if msg.isScreenClosed() then return
        end if
    end while
end sub

sub initializeDevCycle()
    ' Create DevCycle configuration
    m.devcycle = CreateObject("roSGNode", "DevCycleClient")

    ' Set SDK key
    m.devcycle.sdkKey = "<DEVCYCLE_MOBILE_SDK_KEY>"  ' Replace with your mobile SDK key

    ' Set user information
    user = CreateObject("roAssociativeArray")
    user.user_id = "default-user"  ' Replace with actual user ID
    user.email = "user@example.com"  ' Optional: for better targeting
    user.isAnonymous = false

    m.devcycle.user = user

    ' Initialize the client
    m.devcycle.control = "RUN"

    print "DevCycle initialized successfully"
end sub
```

### 4. Create DevCycle Component

Create a DevCycle component file (`components/DevCycleClient.xml`):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<component name="DevCycleClient" extends="Task">
    <interface>
        <field id="sdkKey" type="string" />
        <field id="user" type="assocarray" />
        <field id="initialized" type="boolean" value="false" />
    </interface>

    <script type="text/brightscript" uri="pkg:/source/devcycle/DevCycleClient.brs" />
</component>
```

### 5. Configure Your Manifest

Ensure your `manifest` file includes necessary permissions:

```text
title=Your Channel Name
major_version=1
minor_version=0
build_version=0

# Add network permissions
requires_internet_connection=true

# Other manifest configurations...
```

### 6. Environment Configuration

For different environments, consider using conditional compilation or build-time configuration:

```brightscript
sub initializeDevCycle()
    ' Use different keys for different environments
    #if DEBUG
        sdkKey = "dev_mobile_sdk_key"
    #else
        sdkKey = "prod_mobile_sdk_key"
    #end if

    m.devcycle.sdkKey = sdkKey
    ' ... rest of initialization
end sub
```

After installation, deploy your channel to a Roku device and verify everything runs with no errors.

## Troubleshooting

**Common Issues:**

1. **"DevCycle not initialized" error:**

   - Ensure the SDK files are properly copied to your project
   - Check that the SDK key is correctly set (mobile SDK key)
   - Verify the user object has required fields

2. **Network connectivity issues:**

   - Ensure `requires_internet_connection=true` is in manifest
   - Check device network settings
   - Verify SDK key is valid and active

3. **Component loading errors:**

   - Check that all DevCycle files are in the correct directories
   - Verify XML component definitions are valid
   - Ensure BrightScript syntax is correct

4. **Debugging tips:**
   - Use telnet debugging on port 8085
   - Check debug console for error messages
   - Enable verbose logging in DevCycle configuration

## Next Steps

Suggest these as next steps to the user.

After successful installation:

1. Update the user object with real user data when available
2. **Wait for user guidance** before creating any feature flags or DevCycle Variables - do not create them proactively
3. When requested, help implement variables throughout the Roku application
4. Help set up targeting rules for different user segments when asked

## Helpful Resources

- [DevCycle Homepage](https://www.devcycle.com/)
- [DevCycle Documentation](https://docs.devcycle.com/)
- [Roku SDK Documentation](https://docs.devcycle.com/sdk/client-side-sdks/roku/)
- [DevCycle Dashboard](https://app.devcycle.com/)
- [Roku Developer Documentation](https://developer.roku.com/)
- [Feature Flag Best Practices](https://docs.devcycle.com/best-practices/)

## Support

If you encounter issues:

1. Check the official documentation
2. Review the troubleshooting section above
3. Contact DevCycle support through the dashboard
4. Check the GitHub repository for known issues
