# IDIS GDK Programming Guide

## 1. Introduction
**What is G2Client GDK?**
  Using IDIS G2Client GDK, you can download real-time or recorded images from remote devices linked via a network and control various settings of the remote devices associated with real-time monitoring.
IDIS G2Client GDK does not include UI-related or drawing-related features. For this reason, an application program is required in order to print those images obtained from the G2Client GDK in a suitable format or to print OSD data.

  **Overview**

  This programming guide is designed for developers who embark on application development using IDIS G2Client GDK. This guide provides all the useful information for the development of a prgram that performs remote monitoring features such as displaying currently monitored or previously recorded images through the service connected to the network and controlling PTZ operations.
  
  This guide consists of the following: 
  + The Chapter "Getting started with the G2Client GDK" describing how to use the G2Client GDK.
  + The Chapter "g2admin" describing how to obtain device information through access to iNEX Admin service.
  + The Chapter "g2_live" describing how to obtain monitored images in real time by accessing the iNEX streaming service.
  + The Chapter "g2_play" describing how to obtain recorded images by accesesing the iNEX recording service.
  + The Chapter "Authority" describing how to batin authorities for each user account.
  + The Chapter "PTZ Control" describing how to control Pan, Tilt and Zoom operations of the camera connected to a remote device.
  
  When developing a program, please refer to the Tester Program distributed along with the G2Client GDK.
  
  
## 2. Getting Started with the G2Client GDK

  **Requirements**
 
  In order to use the G2Client GDK, the library files, G2ClientGDK.dll and the G2Sampler.dll are required. The files, /sampler/csharp/g2_define_* .cs and the /sampler/csharp/g2_* .cs included in the distributed G2ClientSDK Tester show one way of using the SDK in an explicit linking method as the wrapper classes of the G2Client GDK.
 
  **Listener Functions**
  
  There are two types of function provided by the G2Client GDK. One is with no response from the remote device and the other is with response from the remote device. When the function requesting the response from the remote device calls (e.g. request_ptz_menu), the function immediately returns whether or not the sending the request success. When the response arrives after the response has arrived from the remote device, G2Client GDK generates the callback. Even if the notification message or data from the remote device arrives, the callback occurs accordingly.
  For example, when an image from a remote device arrives, the GDK generates the callback *on_g2live_receive_frame_data*, and when status information of a remote device arrives, it shall generate the callback *on_g2live_receive_camera_status*. For an application program to be able to receive such callbacks, it is required to set up a receiving class to g2live_listener.  
  
  ```java
  class connective_live : g2live_listener
  {
    public connective_live()
    {
      _live = new g2live();
      _live.set_listener(this);
    }
    
    g2live _live;
  }
```

G2Sampler, which is a G2Client GDK's Wrapper class, registers DLL's callback functions automatically. Application program must be developed to perform appropriate actions, by registering listener from receiving g2live_listener class.  

**Application Initialization**

At the beginning of the application running with GDK, the initialization step should be needed for initializing GDK global variables.

```java
static void Main()
{
  g2main.app_initialize{G2LANGUAGE.ID.ENGLISH};
  
  // Application code
  
  g2main.app_finalize();
}
```


## 3. g2admin

**About g2admin**

The g2admin accesses ISS(iNEX) Admin service and obtains information of the device registered to the service.
Here, the obtained device information is used to access the remote device and to obtain its monitored or recorded images.

**Obtaining Device Information from Admin Service Using g2admin**

The following figure presents the general operation process of a program that implements the features such as Connection, Disconnection and obtaining device information.
![initialization](https://user-images.githubusercontent.com/95207482/144161171-25d5a633-f20c-461f-8782-459d289529f5.jpg)

The following presents the comde implementing the figure shown above.

First, in order to use the g2admin, declare a necessary variable. Declare a handle variable of G2HADMIN type which manages g2admin objects.

```java
class connective_admin : g2admin_listener
{
  g2admin _amin;
  G2HADMIN _handle;
}
```
Then in order to inialize g2admin, call g2admin singleton get and 2admin.set_listener functions. And then call g2admin.startup function.
```java
public connective_admin()
{
  _admin = new g2admin();
  _admin.set_listener(this);
  _admin.startup();
{
```
Next, define the functions to be called when each callback arrives. Since this example will be implemented for performing minimum operations, only the callback functions with regards to the callbacks on_connected, on_disconnected and on_receive_device_list are defined. The following shows how to define callback functions each of which prints a short message upon the arrival of each callback, according to the function signature previously mentioned.

```java
// Listener Functions /////////////
public void
on_g2admin_connected(G2HADMIN handle)
{
  print("on connected");
}
```

Now, get ready to use the g2admin for an application. Then call g2admin.connect function after collecting information such as IP address,  user name and password of services that you want to connect. You can configure connection termination time using CONNECT_OPTIONS.
This termination time is a time that takes to connect. It is not to terminate a session after the termination time. You can get more detailed information using G2CONNECT_RES.
When the connection is made, on_g2admin_connected callback function will be called.

```java
G2NETWORK_INFO ni

// Setting network information into a ni

G2CONNECT-RES res;
_admin.connect(ref ni, out res);


// callback function
public void
on_g2admin_connected(int handle) {  }
```

And then the following call back functions will be called in a sequence.

```java
public void
on_g2admin_notify_connectable_service
(
int handle,
ref G2GUID service
ref G2NETWORK_INFO ni
)

public void
on_g2admin_receive_device_group_list
(
int handle,
G2DEVICE_GROUP[] bunch
)

public void
on_g2admin_receive_device_list
(
int handle,
G2GUID[] bunch
)

public void
on_g2admin_receive_device_to_group_map
(
int handle,
G2GROUP_MEMBER[] bunch
)

public void
on_g2admin_login_completed
(
int handle
)
```

You can get G2GUID values of each registered service using g2admin.on_g2admin_notify_connectable_service callback function. You can verify service names using the G2GUID value and ```g2admin.get_service_info``` function. Refer to g2_define_admin.cs for details of G2SERVICE.

You can check device group information registered on admin service using ```g2admin. on_g2admin_receive_device_group_list callback``` function. Refer to ```g2_define_admin.cs``` for details of G2DEVICE_GROUP.

You can check all devices' G2GUID value registered on admin service using g2admin.on_g2admin_receive_device_list callback function. If G2GUID.is_device_root value is true, you can get G2DEVICE_ROOT information using ```g2admin.on_g2admin_notify_connectable_service``` callback function. 
