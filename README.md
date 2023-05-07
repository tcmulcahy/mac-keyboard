# mac-keyboard

You can query for keyboards with:
```
system_profiler -json SPUSBDataType SPBluetoothDataType | jq '.SPBluetoothDataType[] | to_entries[] | .value | arrays | .[] | to_entries[] | select(.value.device_minorType == "Keyboard") | {name: .key, productID: .value.device_productID, vendorID: .value.device_vendorID}'
```

You can set keyboard-specific mappings by creating a file like the following. See
https://www.digihunch.com/2022/11/key-mapping-on-external-pc-keyboard-on-macbook/
for details.
```
<!--
  Put this file in ~/Library/LaunchAgents/com.example.KeyRemapping.plist to
  automatically remap your keys when macOS starts.
  See https://developer.apple.com/library/archive/technotes/tn2450/_index.html for
  the key "usage IDs". Take the usage ID and add 0x700000000 to it before putting it
  into a source or destination (HIDKeyboardModifierMappingSrc and
  HIDKeyboardModifierMappingDst respectively).
-->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>ca.mulcahy.KeyRemapping</string>
    <key>LaunchEvents</key>
    <dict>
        <key>com.apple.iokit.matching</key>
        <dict>
            <key>com.apple.bluetooth.hostController</key>
            <dict>
                <key>IOProviderClass</key>
                <string>IOBluetoothHCIController</string>
                <key>idProduct</key>
                <!-- TODO: replace with your productID -->
                <integer>024F</integer>
                <key>idVendor</key>
                <!-- TODO: replace with your vendorID -->
                <integer>05AC</integer>
                <key>IOMatchLaunchStream</key>
                <true/>
            </dict>
        </dict>
    </dict> 
    <key>ProgramArguments</key>
    <array>
        <string>/usr/bin/hidutil</string>
        <string>property</string>
        <string>--matching</string>
        <!-- TODO: replace with your productID/vendorID -->
        <string>{"ProductID":0x024F,"VendorID":0x05AC}</string>
        <string>--set</string>
        <!--
          Swap ESC with backtick, and swap caps with left-ctrl.
          See https://developer.apple.com/library/archive/technotes/tn2450/_index.html#//apple_ref/doc/uid/DTS40017618-CH1-TNTAG8
          ESC = 0x29
          backtick = 0x35
          left-ctrl = 0xe0
          caps = 0x39
        -->
        <string>{"UserKeyMapping":[
          {
            "HIDKeyboardModifierMappingSrc": 0x700000029,
            "HIDKeyboardModifierMappingDst": 0x700000035
          },
          {
            "HIDKeyboardModifierMappingSrc": 0x700000035,
            "HIDKeyboardModifierMappingDst": 0x700000029
          },
          {
            "HIDKeyboardModifierMappingSrc": 0x7000000e0,
            "HIDKeyboardModifierMappingDst": 0x700000039
          },
          {
            "HIDKeyboardModifierMappingSrc": 0x700000039,
            "HIDKeyboardModifierMappingDst": 0x7000000e0
          },
        ]}</string>
    </array>
    <key>RunAtLoad</key>
    <true/>
</dict>
</plist>
```

Then run
```
launchctl load <path-to-plist-file>
```
