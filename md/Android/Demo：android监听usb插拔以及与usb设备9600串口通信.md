1、注册usb设备权限管理广播  
```
public static final String ACTION_USB_PERMISSION = “com.android.example.USB_PERMISSION”;  
private PendingIntent mPermissionIntent;  
private BroadcastReceiver permissionReceiver = new BroadcastReceiver() {  
@Override  
public void onReceive(Context context, Intent intent) {  
if (intent.getAction() == ACTION_USB_PERMISSION) {  
UsbDevice device = (UsbDevice) intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);  
if (intent.getBooleanExtra(UsbManager.EXTRA_PERMISSION_GRANTED, false)) {  
if (device != null) {  
connect(device);  
}  
} else {  
Toast.makeText(MainActivity.this, “no permission”, Toast.LENGTH_SHORT).show();  
}  
}  
}  
};


//onCreate
mPermissionIntent = PendingIntent.getBroadcast(this, 0, new Intent(ACTION_USB_PERMISSION), 0);
//注册USB设备权限管理广播
IntentFilter filter = new IntentFilter(ACTION_USB_PERMISSION);
registerReceiver(permissionReceiver, filter);
```

2、注册usb插拔广播接收  
```
public class USBReceiver extends BroadcastReceiver {  
@Override  
public void onReceive(Context context, Intent intent) {  
UsbDevice device = intent.getParcelableExtra(UsbManager.EXTRA_DEVICE);  
Message message = Message.obtain();  
message.obj = device;  
if (intent.getAction() == UsbManager.ACTION_USB_DEVICE_ATTACHED) {  
message.what = MainActivity.WHAT_DEVICE_ATTACHED;  
} else {  
message.what = MainActivity.WHAT_DEVICE_DETACHED;  
}  
MainActivity.getHandler().sendMessage(message);  
}  
}

在onResume和onPause中注册、注销广播  
Private USBReceiver receiver;

//onCreate  
receiver = new USBReceiver();

//onResume  
IntentFilter filter = new IntentFilter();  
filter.addAction(UsbManager.ACTION_USB_DEVICE_ATTACHED);  
filter.addAction(UsbManager.ACTION_USB_DEVICE_DETACHED);  
registerReceiver(receiver, filter);

//onPause  
unregisterReceiver(receiver);
```

3、设备插入后可以读取设备属性  
```
UsbDevice device = (UsbDevice) msg.obj;  
if (device.getProductName().startsWith(“FT232R”)) {  
UsbManager usbManager = (UsbManager) getSystemService(Context.USB_SERVICE);  
if (usbManager.hasPermission(device)) {  
connect(device);  
} else {  
usbManager.requestPermission(device, mPermissionIntent);  
}  
}  
textStatus.append(“in——————\n”);  
textStatus.append(“getProductId:” + device.getProductId() + “\n”);  
textStatus.append(“getProductName:” + device.getProductName() + “\n”);  
textStatus.append(“getDeviceName:” + device.getDeviceName() + “\n”);  
textStatus.append(“getDeviceId:” + device.getDeviceId() + “\n”);  
textStatus.append(“getManufacturerName:” + device.getManufacturerName() + “\n”);  
textStatus.append(“getSerialNumber:” + device.getSerialNumber() + “\n”);  
textStatus.append(“getVersion:” + device.getVersion() + “\n”);  
textStatus.append(“getVendorId:” + device.getVendorId() + “\n”);  
int count = device.getInterfaceCount();  
textStatus.append(“getInterfaceCount:” + count + “\n”);  
for (int i = 0; i < count; i++) {  
UsbInterface usbInterface = device.getInterface(i);  
textStatus.append(“\t\tdevice.getInterface:” + i + “\n”);  
textStatus.append(“\t\tusbInterface.getName:” + usbInterface.getName() + “\n”);  
textStatus.append(“\t\tusbInterface.getId:” + usbInterface.getId() + “\n”);  
int endCount = usbInterface.getEndpointCount();  
textStatus.append(“\t\tusbInterface.getEndpointCount:” + endCount + “\n”);  
for (int j = 0; j < endCount; j++) {  
UsbEndpoint endpoint = usbInterface.getEndpoint(j);  
textStatus.append(“\t\t\t\tusbInterface.getEndpoint:” + j + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getAddress:” + endpoint.getAddress() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getAttributes:” + endpoint.getAttributes() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getDirection:” + endpoint.getDirection() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getEndpointNumber:” + endpoint.getEndpointNumber() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getInterval:” + endpoint.getInterval() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getMaxPacketSize:” + endpoint.getMaxPacketSize() + “\t”);  
textStatus.append(“\t\t\t\tendpoint.getType:” + endpoint.getType() + “\t”);  
}  
}
```

4、发送数据  
```
UsbManager usbManager = (UsbManager) context.getSystemService(Context.USB_SERVICE);  
if (usbManager == null) {  
Message message = Message.obtain();  
message.what = MainActivity.WHAT_MSG;  
message.obj = “usbManager == null”;  
MainActivity.getHandler().sendMessage(message);  
return;  
}  
boolean permission = usbManager.hasPermission(usbDevice);  
if (!permission) {  
Message message = Message.obtain();  
message.what = MainActivity.WHAT_MSG;  
message.obj = “!permission”;  
MainActivity.getHandler().sendMessage(message);  
return;  
}  
UsbDeviceConnection connection = usbManager.openDevice(usbDevice);  
if (connection == null) {  
Message message = Message.obtain();  
message.what = MainActivity.WHAT_MSG;  
message.obj = “connection == null”;  
MainActivity.getHandler().sendMessage(message);  
return;  
}  
UsbInterface usbInterface = usbDevice.getInterface(0);  
UsbEndpoint usbEndpoint = usbInterface.getEndpoint(0);  
connection.claimInterface(usbInterface, true);  
byte[] bytes = new byte[16];  
bytes[0] = 0x11;  
bytes[1] = 0x11;  
bytes[2] = 0x11;  
bytes[3] = 0x11;  
bytes[4] = 0x11;  
bytes[5] = 0x11;  
bytes[6] = 0x11;  
bytes[7] = 0x11;
    bytes[8] = 0x11;
    bytes[9] = 0x11;
    bytes[10] = 0x11;
    bytes[11] = 0x11;
    bytes[12] = 0x11;
    bytes[13] = 0x11;
    bytes[14] = 0x11;
    bytes[15] = 0x11;
    while (true) {
        if (msg != null && msg.length() > 0) {
            connection.bulkTransfer(usbEndpoint, bytes, bytes.length, 0);
            msg = null;
        }
    }
```

​