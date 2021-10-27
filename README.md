<h1><p align="center">VA Product description & Development guidance</p></h1> 

## What is VA? ##
VirtualAPP (abbreviation: VA) is a sandbox product running on Android system, which can be understood as a lightweight "Android virtual machine". Its product form is a highly extensible, customizable, integrated SDK that allows you to develop a variety of seemingly impossible projects based on or using VA. Now, VA is widely used in in many technology fields as following: mini game collection, blockchain, cloud control, silent hot fix and so on. On the one hand, you can realize cloud control mobile office security and achieve military and government data isolation with VA. On the other hand, you can implement script automation, device-info-mock, and plug-in development. Meanwhile, you can realize multi space and games booster. You can also rent the mobile game account and use the mobile controller without activation by VA. <br> **The code on Github has stopped updating in December 2017. The code of commercial version is continuously being updated. If you need license to obtain the latest code, please contact WeChat: 10890.**


## Terminology in VA ##
Terminology | Explanation
---- | ---
Host | The APP that integrates the VirtualAPP class library (lib) is called a host.  
Host Plug-in | A host package is used to run another ABI on the same phone. It also called plug-in package, extension package, host plug-in package, host extension package.
Virtual APP / VAPP | Multi space in VA's virtual environment
External APP | APP installed in the real environment of the mobile phone
<br/>

## VA Technical architecture ##
![](https://cdn.jsdelivr.net/gh/xxxyanchenxxx/temp@1.0/doc/va_architecture.jpg)  
VA technology involves the APP layer, Framework layer and Native layer of Android in total.
An APP wants to run on Android, the system will accept it after installing. The APP installed inside the VA is not actually installed into the system, it can't run under the normal circumstance. So how to get it to run?
Answer: The only way to do this is to "trick" the system into thinking it has been installed. This "deception" process is the core work of the VA Framework, and is also the core technical principle of the VA.  

**Here is the description of what did each layer do:**

Layer | Main work
---- | ---
VA Space | An internal space is provided by the VA for the installation of the APP to be run inside it, and this space is system isolated.
VA Framework | This layer is mainly a proxy for Android Framework and VAPP, which is the core of VA. And VA provides a set of VA Framework of its own, which is between Android Framework and VA APP. </br>1. For VAPP, all the system services it accesses have been proxied by VA Framework, which will modify the request parameters of VAPP and send all the parameters related to VAPP installation information to Android Framework after changing them to the parameters of the host （Some of the requests will be sent to their own VA Server to be processed directly, and no longer send to the Android system）. This way Android Framework receives the VAPP request and checks the parameters, and it will think there is no problem.</br>2. When the Android system finishes processing the request and returns the result, the VA Framework will also intercept the return result and restore all the parameters that have been original modified to those that were sent during the VAPP request. This way the interaction between VAPP and Android system can work.
VA Native | The main purpose of this layer is to accomplish 2 tasks: IO redirection and the request modification for VA APP to interact with Android system. </br>1. IO redirection is some APPs may be accessed through the hard code absolute path. But if the APP is not installed to the system, this path does not exist. Through IO redirection, it will be redirected to the path to install inside VA.</br>2. In addition, there are some jni functions that cannot be hooked in VA Framework, so they need to be hooked in the native layer.
</br>

In summary:
As you can see from the above technical architecture, the internal VA APP actually runs on top of VA's own VA Framework. VA has intercepted all system requests from its internal APP, and through this technology it can also have full control over the APP, not just the multi space. And for the convenience of developers, VA also provides SDK and Hook SDK.


## VA Process architecture#
![](https://cdn.jsdelivr.net/gh/xxxyanchenxxx/temp@1.0/doc/va_process.jpg)    
There are five types of processes in the VA’s runtime: CHILD process, VA Host Main process, VA Host Plugin process, VAPP Client process, and VAServer process. 
To support both 32-bit and 64-bit APPs, VA needs to install two packages: a master package and a plug-in package ( In this document, the main package is 32 bits and the plug-in package is 64 bit ).
Two packages are also necessary because a package can only run in one mode, either 32-bit or 64-bit. So for 32-bit APPs, VA uses the 32-bit main package to run, and for 64-bit APPs, VA uses the 64-bit plug-in package to run.
The main package contains all the code of VA, and the plug-in package contains only one piece of code that loads the main package code for execution, no other code. So plug-in package rarely needs to be updated, just the main package. 
In addition, whether the main package is chosen to use 32-bit or 64-bit can be modified in the configuration file ( For example, for users who want to access GooglePlay, it will be modified to 64-bit for the main package and 32-bit for the plug-in package ).

**The functions and explanations of the each type of process are as follows:**</br>

Process Type | Function
---- | ---
CHILD | Other processes integrated by VA Host, such as: keepalive process, push process, etc.
VA Host Main | The process where the UI main interface of the VA main package is located. The default main package is 32-bit and the plug-in package is 64-bit, which can be modified and switched in the configuration file
VA Host Plugin | The process that supports the plug-in package of 64-bit APP. The default main package is 32-bit and the plug-in package is 64-bit, which can be modified and switched in the configuration file.
VAPP Client | The process generated by the APP installed into VA after it starts, it will modify io.busniess.va:pxxx process name to the real process name of VAPP when it runs.
VAServer | The process where the VA Server is located, it is used to handle requests in VA that are not assigned to the system for processing, such as APP installation processing.
<br/>

## VA can satisfy almost all your needs ##
Through the above technical architecture, we can know that VA can fully control APP and provide Hook SDK, which can satisfy almost all your needs in various fields: 

1. Satisfy the need of** double/multi space**   
VA allows you to install multiple WeChat/QQ/WhatsAPP/Facebook and other APPs on the same mobile phone, so you can have one phone with multiple accounts logged in at the same time.  

2. Satisfy the need of** mobile security**  
VA provides a set of internal and external isolation mechanisms, including but not limited to (file isolation / component isolation / process communication isolation). Simply speaking, VA internal is a "completely independent space". 
Through VA, work affairs and personal affairs can be safely separated without mutual interference. With a little customization, you can achieve mobile security-related needs such as application behavior audit, data encryption, data acquisition, data leakage prevention, anti-attack leaks and so on.    
    **2.1 Application behavior audit**  
The HOOK capability provided by VA can realize real-time monitoring of user usage behavior and upload violation information to the server. And it's easy to implement things like Time Fence ( whether a feature of the APP can be used in a certain time ), Geo Fence ( whether a feature of the APP can be used in a certain area ), sensitive keyword filtering interception and other functional requirements.
	**2.2 Data encryption**  
The HOOK capability provided by VA can realize all data/file encryption of the application, ensuring data/file landing security.

	**2.3 Data acquisition**  
The HOOK capability provided by VA can realize the demand for real-time silent upload of application data, such as chat records and transfer records, preventing them from being deleted afterwards without traceability.  
	**2.4 Data leakage prevention**  
The HOOK capability provided by VA can realize application anti-copy/paste, anti-screenshot/recording, anti-sharing/forwarding, watermark traceability and other requirements.   
	**2.5 Anti-attack leaks**  
With the application control capability provided by VA, privacy-related behaviors such as SMS/ address book/call log/ background recording/background photo/ browsing history and location information can be completely controlled in sandbox, prevent Trojan horses/malicious APPs from acquiring users' real private data, causing serious consequences such as leakage of secrets.
3. Satisfy the need of** ROOT without HOOK **  
VA provides Hook capability of Java and Native. With VA, you can easily achieve functions required by various scenarios, such as virtual positioning, changing device, APP monitoring and management, mobile security and so on.  

4. Satisfy the need of** silent installation **  
VA provides the ability to silently install, silently upgrade and silently uninstall APPs. For example, the application store or game center can be integrated with VA to avoid the need for users to manually click to confirm the installation operation, so that it can be installed into VA immediately after downloading, bringing users an experience like "small program" , completely avoiding the problem of applications not easily installed by users.  

5. Satisfy the need of** APP controlled ** 
You can clearly grasp the system API, sensitive data, device information, etc. accessed by the APP through VA. For example, whether the APP accesses the contacts, photo albums, call log, whether it accesses the user's geographic location and other information.
Of course, you can also control or construct custom messages to these APPs via VA, and not only that, you can also get access to the APP's private data, such as chat database and so on. In a word, through the application control capability provided by VA, you can easily control all the behaviors of the APP, even modify the content of the APP and server interaction and so on .  </br>


6. Satisfy the need of** overseas markets **  
VA implements support for Google services to support overseas APPs running, such as Twitter, Messenger, WhatsAPP, Instagram, FaceBook, Youtube and so on.

7. Satisfy the need of** almost everything you can think of **  
VA has complete oversight and control over the internal APP, and can meet almost any of your needs！
8. 8.VA is also the only commercially licensed product in this technology area   
**Hundreds of** licensed customers are currently paying to use the commercial version of VirtualAPP code, and the APP integrated with VirtualAPP code is launched more than 200 million times per day. Many Android engineers provide us with user feedback in different scenarios, and through our technical team's continuous optimization and iteration, we continue to improve product performance and compatibility.


VA Specialized capabilities
---

- Cloning ability<br/>
You can clone the APP already installed in the external system and run it internally without mutual interference. Typical application scenario is double space.

- Without installation ability<br/>
In addition to cloning already installed, VA can install (externally silent ) apk's directly internally and run them directly internally. Typical application scenarios are plug-in, standalone APP marketplace and so on.

- Double space ability<br/>
VA is not only "double space", but also has a unique multi-user mode that allows users to open the same APP internally for an unlimited number of times.

- Internal and external isolation ability<br/>
VA is a standard sandbox, or "virtual machine", that provides a set of internal and external isolation mechanisms, including but not limited to (file isolation/component isolation/process communication isolation). Simply put, the inside of a VA is a "completely separate space". Simply put, the inside of a VA is a "completely separate space". Based on it, you can realize a "virtual phone" on your cell phone with a little customization. Of course, you can also use your imagination to customize it for data encryption, data isolation, privacy protection, and enterprise management applications.

- Full control over internal APPs ability<br/>
VA has complete monitoring and control over the internal APP, which is absolutely impossible to achieve in an external environment without Root.

<details>
<summary>Details(Drop down to open)</summary>
  1. Service request control. First, VA directly provides some service request interception, you can easily customize these service requests when integrating VA, including but far from limited to (APP request to install apk / APP request to open certain files / APP request for location data / APP request for phone information, etc.)<br/><br/>
  2. System API control. VA virtualizes and implements the entire Android system framework, which is the principle that VA can run apk internally without installation. And you can through modify the virtual framework's implementation to dynamically monitor and analyze the behavior of the app, etc. In addition, you can also mock some system behavior to achieve some needs that are difficult to achieve externally (e.g. game controller).<br/><br/>
  3. Memory read and write. VA can read and write the memory of internal APP processes without Root.<br/><br/>
  4. Root without debugging. VA can debug (ptrace) internal APP processes without Root, based on which you can also achieve Root-free process injection.<br/><br/>
  5. Load arbitrary "plug-in" and "behaviors". The APP process inside VA is derived from the Client side code of the VA framework, so you can insert any "load" and "control" logic into the entry code of the process. These are very simple to implement.<br/><br/>
  6. Hook. VA has a set of built-in Xposed framework and native hook framework running on all versions of Android (until AndroidQ), based on it, you can easily Hook any Java/Native of any internal APP.<br/><br/>
  7. File control. VA built in a complete file redirection, which allows easy control of reading and writing of files from internal apps. Based on it, you can realize many functions such as protection and encryption of files can be achieved.<br/><br/>
  8. Note: The above control capabilities are implemented with code or examples for reference.
</details>


VA Other features
---

- High performance<br/>
Process-level "virtual machine", VA's unique implementation model makes its performance almost the same as that of the native APP, and does not need a long startup of ordinary virtual machines.

- Full version support<br/>
Support 5.0-12.0, 32-bit/64-bit APP, ARM and X86 processor. And support Android version in the future which will be updated.

- Easy Expansion and Integration<br/>
The integration of VA is similar to the normal Android library, even if your APP has been online, you can conveniently integrate VA and enjoy the capability brought by VA.

- Support Google services<br/>
Provide support for Google services in order to support overseas APPs.


## Comparison between VA and other technical solutions ##
When doing enterprise-level mobile security, it is often necessary to control the APP, and the following is a comparison of possible technical solutions listed： 

Technical solution | Principle introduction | Comment |  Running performance | Compatibility stability | Project maintenance cost
---- | --- | ---  | ---  | ---  | --- 
Repackage | Repackage the target APP by decompiling it and adding your own control code | 1. Nowadays, almost all APPs have hardened or tamper-proof protection, and repackaging is already a very difficult task</br> 2.The mobile phone system will also detect whether the APP is repackaged, if it is repackaged, it will directly prompt the user that there is a security risk, and even not allow the installation</br>3.For each APP, even each version to go deep to reverse analysis, time-consuming and difficult to maintain  | Excellent  | Poor  | High
Custom ROM | By customizing the system source code and compiling it to flash to the designated mobile phone | Only for specified internal mobile phones, too limited to be extended  | Excellent  | Excellent  | High
ROOT the mobile phone | By rooting the mobile phone，flashing a framework which is similar to Xposed | 1.Now, root the mobile phone is an unlikely thing</br> 2.In reality, it is difficult for users to root their own mobile phones  | Excellent  | Poor  | High
VA | Lightweight virtual machine with high speed and low device requirements | No risk point mentioned above  | Excellent  | Excellent. Hundreds of companies testing feedback at the same time  | Low. 
VA provides API and a professional technical team to ensure the stable operation of the project
<br/>
As you can see from the above comparison, VA is an excellent product and can reduce your development and maintenance costs.

## Integrating VA Steps ##
Step 1: Call the VA interface```VirtualCore.get().startup()```in your application to start the VA engine.  
Step 2: Call VA interface```VirtualCore.get().installPackageAsUser(userId, packageName)```to install the target APP into VA.
Step 3: Call VA interface```VActivityManager.get().launchApp(userId, packageName)```to start the APP.   
**With only the above 3 APIs to complete the basic use, VA has shielded the complex technical details and provided the interface API to make your development easy.**

## VA compatible stability ##
VA has been extensively tested by ** hundreds of **companies, including **high standards of testing and feedback of dozens of listed companies**, covering almost all types of equipment and scenarios at home and abroad, providing full protection for your stable operation!
 

Up to now, the supported system versions:

System version | Whether to support
---- | ---
5.0 | support
5.1 | support
6.0 | support
7.0 | support
8.0 | support
9.1 | support
10.0 | support
11.0 | support
12.0 | support
<br/>


Supported App Types:

App Type | Whether to support
---- | ---
32-bit APP | support
64-bit APP | support
<br/>

Supported HOOK Types:

Hook Type | Whether to support
---- | ---
Java Hook | support
Native Hook | support

Supported CPU Types:

Hook Type | Whether to support
---- | ---
ARM 32 | support
ARM 64 | support
<br/>

## How to give feedback on problems encountered with integrated VA ? ##
After the purchase of the license we will establish a WeChat group, any problems can always feedback to us, and according to the priority in the first time to deal with.

## VA Development document ##
Please refer to the VA development documentation：[Development document](https://github.com/asLody/VirtualApp/blob/master/doc/VADev.md)



License Instructions
------
VirtualApp virtual machine technology belongs to: Jining Luohe Network Technology Co., LTD. It applied for several VirtualApp intellectual property rights from 2015 to 2021 and` is protected by the Intellectual property Law of the People's Republic of China`.When you need to use the code on Github,**please purchase a commercial license**，and receive the full source code of the latest VirtualApp commercial version.Hundreds of licensed customers are paying to use the commercial version of VirtualApp code, and the app integrated with VirtualApp code is launched more than 200 million times a day. Many Android engineers provided us with user feedback in different scenarios, and through our technical team's continuous optimization and iteration, VirtualApp Commercial Edition code has better performance and higher compatibility. `The company of that year will become one of them after obtaining the license, and enjoy the technological achievements after the continuous iteration. And we can interact and collaborate with our licensed customers operationally, technically and commercially.`

<br/>
Person in charge: Mr. Zhang <br/>
Telephone: +86 130-321-77777 <br/>
WeChat：10890 <br/>
<br/>


Serious statement
------
If you use VirtualApp for **internal use, commercial profit or upload it to the application market**without licensing. We will take evidence and then report you to the police (for copyright infringement) or prosecute you. It will cause your company to undertake criminal liability and legal action, and affect your company's goodwill and investment.`Purchasing a commercial license can save you a lot of time developing, testing and refining compatibility, leaving you more time for innovation and profitability.`Luo He Technology has called to the police and sued a number of individuals and companies in 2020.<br/>

**In response to the national call for the protection of intellectual property rights! Anyone who reports that his or her company or other companies are using VirtualApp code to develop products without licensing will be given a cash reward upon verification. We will keep the identity of the whistleblower confidential! Reporting WeChat: 10890.**

  <br/>

Major updates of the commercial version
------

1. Compatible with the latest Android S
2. Not easily misreported by anti-virus software
3. Framework optimization, performance greatly improved
4. Mobile system and APP compatibility greatly improved
5. Run Google services perfectly
6. Supports running pure 64-bit Apps
7. Built-in `XPosed Hook` framework
8. Add positioning mock code
9. Add code to change device 
10. Nearly 400 other fixes and improvements, please see the following table for detail

<br>

2017 - 2021 Commercial Edition Code Update Details
------

**September 21, 2021 to October 10, 2021 Commercial Code Updates**

356、Fix the GMS login problem on 11.0<br/>
355、Fix 11.0 some APP read and write sdcard error problem<br/>
354、Fix the problem that APP may not open after the death of VA core process<br/>
353、Add the error log that can't start when no plug-in is installed<br/>


**August 22, 2021 to September 20, 2021 Commercial Code Updates**

352、Horizontal screen re-adaptation<br/>
351、Fix the problem that some APPs cannot be opened after installation through file protocol<br/>
350、Fix the problem of Intent data loss in the Intent passed to JobIntentService<br/>
349、Fix the problem that the second call of JobIntentService does not work<br/>
348、Fix the problem of crashing some APPs on Huawei cell phones<br/>
347、Fix the game login problem on Xiaomi phone<br/>
346、Fix the problem that some applications cannot be opened after reinforcement<br/>
345、Add detection of associated start permission<br/>
344、targetSdk 30适配<br/>
343、Fix the problem that some applications can't access the Internet when targetSdk is 30<br/>
342、Fix the problem that sdcard can't be accessed when targetSdk is 30<br/>
341、Use cmake to replace gradle task in compile script.<br/>
340、Remove obsolete documents<br/>

<details>
<summary>December 2017 to August 21, 2021 Commercial code updates (Drop down to open)</summary>
	
**August 7, 2021 to August 21, 2021 Commercial Code Updates**

349、Tweak and optimize gradle script<br/>
348、hidedenApiBypass support for Android R+<br/>
347、targetSdk 30 support<br/>
346、Fixthe bug that VIVO system service<br/>
345、Fix the bug that VIVO phone can't use camera<br/>
344、Fix dex loading abnormal state acquisition<br/>
343、Fix libart.so path problem on Android R<br/>
342、Fix the bug of Andoid Q+ delete notification<br/>
341、Fix the permission check of APN uri<br/>
340、Fix Android R suspend resume thread state<br/>
339、Fix some hook failure cases in debug mode<br/>
338、Fix some bugs of hook after R<br/>

**April 25, 2021 to August 6, 2021 Commercial Code Updates**

337、Fix the problem that some phones cannot upload avatars in Tan Tan<br/>
336、Fix Android 10 Huawei device IO redirection problem<br/>
335、Adjust the horizontal and vertical screen logic, reduce the occurrence of abnormalities<br/>
334、Add the callback interface of Activity life cycle<br/>
333、Fix the broadcasting problem of Android 12<br/>
332、Fix the bug of abnormal status of some interfaces of WeChat<br/>
331、Fix the support of Outlook, One drive, Teams, Zoom and other overseas APPs.<br/>
330、Fix the bug Android 11 a permission request<br/>
329、Fix the problem that some cocos2d engines only display half screen<br/>
328、Fix the problem that WeChat can not send files under multi-user<br/>
327、split apk support<br/>
326、Android S support<br/>

**February 24, 2021 to April 24, 2021 Commercial Code Updates**

325、Adapt to multi-user environment<br/>
324、Fix the compatibility problem of the new version of WeChat<br/>
323、Compatible with more enterprise level reinforcement<br/>
322、Support VAPP setting power source optimization<br/>
321、Fix missing permission statement<br/>
320、Fix the reference of android.test.base library on Android 11<br/>
319、Optimize ext plugin judgment<br/>
318、Optimize the selection of ABI during installation<br/>
317、Fix Google docs crash on Android 11<br/>

**October 15, 2020 to February 23, 2021 Commercial Code Updates**

316、Solve the compatibility of the new version of Love Encryption, Bang Bang and other reinforcement<br/>
315、Fix the problem of WhatsApp not showing cold boot Splash<br/>
314、Optimize the recognition of system APP<br/>
313、Improve the support in multi-user environment<br/>
312、Solve the problem that ext plug-in is stuck in some cases<br/>
311、Support Google Play to download APP in VirtualAPP<br/>
310、Fix the problem that Android 11 QQ can not display pictures<br/>
309、Compatible with Android 11 running Google Service<br/>
308、Fix the problem that Android 11 can't run chromium<br/>
307、Support Hook @CriticalNative Method<br/>
306、Fix the problem that JDK 13 cannot be compiled and run<br/>
305、Fix the problem that Service may crash in some cases<br/>
304、Fix the problem that Android 11 cannot load private data of external storage<br/>
303、Fix the problem that low version APP cannot use org.apache.http.legacy<br/>
302、Fix the problem that the system task stack only shows the last one in some cases<br/>
301、Improve the build script for different platforms<br/>
300、Fix the problem that Android 11 cannot read obb<br/>
299、Fix the problem that the software is not backward compatible<br/>
298、Rebuild VAPP installation framework<br/>
297、Rebuild virtual file system<br/>
296、Fix the problem that WebView cannot be started under certain circumstances<br/>
295、Fix the bug of VAPP uninstall and reinstall<br/>
294、Fix the mobile game "LOL" login exception problem<br/>
293、Support the installation of Splits APK<br/>
292、Support dynamic configuration of the main package environment<br/>
291、Fix the problem of 32-bit QQ calling 64-bit WeChat delay<br/>
290、Fix the problem of Messenger calling Facebook crash<br/>
289、Optimize the support of Google service framework<br/>
288、Realize the new extension package synchronization mechanism<br/>
287、Fix the exception problem of Android 11 official version<br/>
286、Add system Package cache to optimize performance<br/>
285、Fix the bug that the disabled component can still be queried by PMS<br/>
284、Fix the problem of abnormal Launch behavior in some interfaces of WeChat<br/>
283、Fix the bug that ContentProvider.getCallingPackage returns Host package name<br/>
282、Fix the bug of uid virtualization and solve the problem that some app permission check fails<br/>
281、Rewrite the implementation of PendingIntent, IntentSender<br/>
280、Optimize process management, fix the long-standing probabilistic process deadlock problem<br/>
279、Rewrite Service implementation, Service life cycle more accurate, not easy to be killed<br/>


**September 13, 2020 to October 15, 2020 Commercial Code Updates**

278、Fix the problem that 64-bit APP cannot call 32-bit APP<br/>
277、Fix the problem of loading HttpClient in Android R <br/>
276、Fix the problem of a crash in Android R debug mode<br/>

**August 23, 2020 to September 12, 2020 Commercial Code Updates**

275、Add the missing service hook<br/>
274、Fix the problem that Baidu Translate cannot be started <br/>
273、Fix the problem that the split app downloaded by GP cannot be started<br/>

**July 10, 2020 to August 22, 2020 Commercial Code Updates**

272、Fix Service creation<br/>
271、Added missing Hook for NotificationService<br/>
270、Fix Yotube crash<br/>

**May 19, 2020 to July 9, 2020 Commercial Code Updates**

269、Preliminary adaptation of Android 11 beta1<br/>
268、Fix the problem of multi space flashback in RED<br/>
267、Fix the problem of "Application signature is tampered" reported by  multi space of some APPs<br/>

**April 24, 2020 to May 18, 2020 Commercial Code Updates**

266、Fix sh call error<br/>
265、Fix the problem that Facebook cannot be logged in in the latest version of 9.0 or above<br/>
264、Help Enterprise WeChat to fix the problem that it can't take photos when starting virtual storage<br/>
263、Fix the problem that 64-bit APP can't open Activity in some cases<br/>

**March 24, 2020 to April 23, 2020 Commercial Code Updates**

262、Fix the problem that Vivo device prompts to install game SDK<br/>
261、Fix the problem that Android Q cannot load some system so<br/>
260、Fix Huawei device microblog not responding<br/>
259、Ignore crashes caused by unnecessary permission checks<br/>
258、Fix the crash of WPS sharing files<br/>
257、Flashback issue in some 10.0 devices<br/>

**March 7, 2020 to March 23, 2020 Commercial Code Updates**

256、Fix WeChat open two pages at the same time problem<br/>
255、Fix the problem that WeChat login successfully but return to the login page<br/>
254、Fix the problem that the latest version of QQ can not download attachments<br/>
253、Update SandHook version<br/>
252、Fix the problem of unsigned Apk installed above 9.0<br/>
251、Fix the positioning problem of 10.0<br/>

**January 16, 2020 to March 6, 2020 Commercial Code Updates**

250、Tweak lib redirection logic<br/>
249、Fix crash issue on Samsung 10.0 systems<br/>
248、Fixed hook exception in release build<br/>
247、Added SandHook proguard rules<br/>
246、Fixed compatibility issue with VirtualApk in some APPs <br/>
245、Fixed VA internal request to install apk failed<br/>

**December 26, 2019 to January 15, 2020 Commercial Code Updates**

244、Fix a missing hook in Android Q<br/>
243、Disable AutoFill in Emui10<br/>
242、Add new api to end all activities<br/>

**December 15, 2019 to December 25, 2019 Commercial Code Updates**

241、Fix the problem that enterprise WeChat and other apps cannot be launched on Emui10<br/>
240、Fix a possible crash in 4.x<br/>
239、Upgrade SandHook to fix Hook for Thread class<br/>
238、Fix the permission problem caused by some interfaces of Android Q<br/>

**November 20, 2019 to December 14, 2019 Commercial Code Updates**

237、Fix crash caused by Notification cache<br/>
236、修复高版本 Notification 的 classloader 问题<br/>

**2019年 11月9号 至 2019年 11月19号 商业版代码更新内容**

235、修复 Android 5.x 的 ART Hook <br/>
234、修复 ART Hook 可能导致的死锁问题 <br/>

**2019年 11月2号 至 2019年 11月8号 商业版代码更新内容**

233、修复 WPS, 网易邮箱等在 Q 设备上崩溃的问题 <br/>
232、修复汤姆猫跑酷在部分 Q 设备上崩溃的问题 <br/>
231、修复 QQ 在部分 Q 设备上崩溃的问题 <br/>

**2019年 10月25号 至 2019年 11月1号 商业版代码更新内容**

230、修复克隆 Google Play 下载的 64位 App<br/>
229、修复企业微信 <br/>
228、修复 Telegram <br/>

**2019年 10月8号 至 2019年 10月24号 商业版代码更新内容**

227、修复 Android P 下 AppOspManager 的异常 <br/>
226、添加 Android P 下 ActivityTaskManager 丢失的 Hook <br/>
225、修复 Android P 下 Activity Top Resume 异常 <br/>
224、支持在系统多用户模式下运行! <br/>

**2019年 10月8号 商业版代码更新内容**

223、修复Android P 以上内部 app 返回桌面异常的问题 <br/>
222、64位分支支持 Android Q <br/>

**2019年 9月20号 至 2019年 10月7号 商业版代码更新内容**

221、修复安装在扩展插件中的 apk 无法正确显示图标和名称的问题 <br/>
220、修复 twitter 无法打开的问题 <br/>
219、正式兼容 Android Q 正式版! <br/>
218、修复 Android Q 某些 Activity 无法再次打开的问题 <br/>
217、初步适配 Android Q 正式版 <br/>
216、修复数个64位分支的 Bug <br/>
215、新增加支持32位插件的64位分支，该分支支持32位旧设备并且64位设备在32位插件的情况下可以支持32位旧应用 <br/>

**2017年 12月 至 2019年 7月 30 日 商业版代码更新内容**

214、改进 App 层提示信息 <br/>
213、改进部分编码 <br/>
212、修复从宿主向插件发送广播的方法 <br/>
211、兼容最新版 gradle 插件 <br/>
210、增加广播命名空间以避免多个使用 VA 技术的 App 互相干扰 <br/>
209、修复 IMO 打不开的问题 <br/>
208、修复部分 ContentProvider 找不到的问题 <br/>
207、支持纯32位模式，以兼容老设备 <br/>
206、初步支持纯64位模式，以应对8月份的谷歌市场的策略变化 <br/>
205、适配到 Android Q beta4 <br/>
204、修复了货拉拉无法安装的问题<br/>
203、优化了64位apk的判定逻辑<br/>
202、修复配置网络证书的 App 的联网<br/>
201、重构组件状态管理<br/>
200、优化 MIUI/EMUI ContentProvider 兼容性<br/>
199、修复 StorageStats Hook<br/>
198、修复快手无法登陆<br/>
197、修复 YY 无法启动，更好的兼容插件化框架<br/>
196、修复 Facebook 登陆<br/>
195、修复 Google Play 下载的 App 无法找到 so 的问题(皇室战争)<br/>
194、修复 split apk 支持<br/>
193、修复 Youtube 无法启动<br/>
192、修复优酷无法启动的问题<br/>
191、修复多开时app间可能存在广播namespace冲突的BUG<br/>
190、采用新的策略绕过Android P以后的Hidden Policy API<br/>
189、适配Android Q(beta1)<br/>
188、修复华为设备部分app无法识别存储的问题<br/>
187、修复启动进程可能失败导致app无法运行的问题<br/>
186、修复4.4设备部分native符号无法找到的问题<br/>
185、修复部分设备WebView包名获取失败的问题<br/>
184、修复Service细节处理的问题<br/>
183、优化启动速度<br/>
182、修复WebView在少数机型加载失败的情况<br/>
181、修复Lib决策的问题<br/>
180、修复部分华为机型无法读取内存卡的问题<br/>
179、修复Service可能存在的问题<br/>
178、允许根据intent判断Activity是否在外部启动<br/>
177、修复部分机型上Gms和Google Play启动到了不正确的环境<br/>
176、修复新实现的StaticBroadcast导致的兼容性问题<br/>
175、修复Android P上无法使用apache.http.legacy的问题<br/>
174、实现Native trace<br/>
173、优化IO Redirect性能<br/>
172、修复wechat部分时候出现网络无法连接的问题<br/>
171、修复小概率process attach不正确的BUG<br/>
170、开始下一阶段的ROADMAP<br/>
169、解决Android P无法注册超过1000个广播导致的问题<br/>
168、修复可能导致ANR的DeadLock<br/>
167、修复部分app动态加载so失败的问题<br/>
166、修复免安装运行环境下部分机型第一次打开出现黑屏的问题<br/>
165、兼容适配多款主流的Android模拟器<br/>
164、优化启动性能<br/>
163、解决多个内存泄露问题<br/>
162、修复IO Redirect优先级的问题<br/>
161、修复8.0以下设备Messenger无网络连接的问题<br/>
160、修复双开时外部app卸载时内部app仍然保留的BUG<br/>
159、修复部分腾讯加固无法运行的问题<br/>
158、修复Instagram无法登录Facebook的BUG<br/>
157、修复进程小概率可能重复启动的BUG<br/>
156、修复GET_PERMISSIONS没有获取权限的BUG<br/>
155、修复startActivityIntentSender的BUG<br/>
154、修复vivo设备部分Activity无法启动的问题<br/>
153、修复app无法调用外部app选择文件的问题<br/>
152、完善Android P的兼容<br/>
151、兼容Android P的Google服务<br/>
150、解决Messenger部分功能异常<br/>
149、完善IO Redirect<br/>
148、大量适配Gms, 修复Gms运行过程中进程无限重启的问题<br/>
147、重新实现Service的运行机制<br/>
146、完善64bit，提供了部分ROM配置64bit Engine权限的API<br/>
145、修复了4.4设备上的Activity启动问题<br/>
144、支持excludeFromRecent属性<br/>
143、修复Instagram无法Facebook登录的问题<br/>
142、修复Facebook第一次登录闪退的问题<br/>
141、支持以64位模式运行Gms、Google play、Play game<br/>
140、支持在双开/免安装运行的Google play中下载和安装app<br/>
139、修复DownloadManager的BUG<br/>
138、修复Google play返回上层时重启界面的BUG<br/>
137、修复免安装模式下so决策问题<br/>
136、优化构建脚本，便于引入项目<br/>
135、修复移动MM SDK无法启动的问题<br/>
134、修复微信摇一摇的BUG<br/>
133、修复中兴设备不稳定的BUG<br/>
132、支持ARM64下的IO Redirect<br/>
131、修复USE_OUTSIDE模式下外部app更新时，内部app没有更新的BUG<br/>
130、兼容最新Android 9.0(代号: pie) 及正式版之前发布的四个Preview版本<br/>
129、兼容内置houdini的x86设备<br/>
128、WindowPreview技术，使app启动与真实app达到一样的速度<br/>
127、新的ActivityStack以提高app运行质量<br/>
126、解决接入Atlas Framework的app运行异常的问题<br/>
125、现在可以定义虚拟app返回桌面的具体行为<br/>
124、现在双开模式下app随系统动态更新，不需要手动检查<br/>
123、支持targetSdkVersion >= 26时仍可正常运行低版本的app<br/>
122、兼容腾讯游戏管家的QDroid虚拟引擎 (beta)<br/>
121、大量重构底层代码，大幅提升运行速度<br/>
120、修复网易新闻分享到微博后无法取消的问题<br/>
119、修复App自定义权限无法识别的问题<br/>
118、修复墨迹天气app无法启动的问题<br/>
117、修复部分政府app无法启动的问题<br/>
116、API的变动详见代码<br/>
115、修复三星系列应用的相互调用问题<br/>
114、修复小米应用在非小米系统的账号问题<br/>
113、修复分享/发送等第三方调用，返回页面不正常<br/>
112、修复应用宝提示不能安装<br/>
111、调用第三方app，对uri进行加密<br/>
110、适配前刘海<br/>
109、适配小米rom的hook<br/>
108、适配努比亚录音问题<br/>
107、内部悬浮窗权限控制<br/>
106、优化自定义通知栏的处理<br/>
105、修复Context的INCLUDE_CODE权限问题<br/>
104、适配华为，oppo的角标<br/>
103、修复百度视频的进程重启问题<br/>
102、修复某些snapchat的无法启动问题<br/>
101、适配autofill服务，例如piexl系列<br/>
100、完善64位的io hook<br/>
99、优化hook库的兼容性，加回dlopen<br/>
98、64位扩展包的so移到32位主包。（jni代码改动后，在Run之前，请先build一次）<br/>
97、通知栏改动：适配8.1的通知渠道；移除应用时，移除应用的全部通知<br/>
96、兼容部分app，需要设置android:largeHeap=true<br/>
95、修复ffmpeg库的视频无法播放问题<br/>
94、优化横竖屏切换<br/>
93、降低通过Intent.ACTION_VIEW调用外部Activity限制。<br/>
92、兼容MG SDK<br/>
91、64位支持还在开发阶段<br/>
90、更新混淆配置app/proguard-rules.pro，必须加规则-dontshrink<br/>
89、优化模拟机型，例如：模拟后，某些app不出现设备验证<br/>
88、提高dex2oat兼容性<br/>
87、优化模拟定位<br/>
86、移除dlopen<br/>
85、targetVersion可以改为26：支持targetVersion<23的app动态权限申请，支持targetVersion<24的文件Uri<br/>
84、installPackage改为默认异步形式<br/>
83、为了支持64位模式，换回aidl<br/>
82、去掉SettingHandler现在可以动态设置特殊规则，规则会存储，不需要重复设置<br/>
81、增加2个native_setup<br/>
80、提高jobService兼容性<br/>
79、ShortcutService相关：关联VASettings.ENABLE_INNER_SHORTCUT<br/>
78、为了稳定性和运行效率，去掉上个版本的蓝牙，wifi，不声明权限的适配。<br/>
77、增加app启动异常的广播Constants.ACTION_PROCESS_ERROR<br/>
76、修复少数游戏横屏判断问题<br/>
75、demo增加机型模拟<br/>
74、适配vivo一个自定义权限（后台弹窗）VA是把一个历史acitivty返回前台，vivo需要这个权限。<br/>
73、如果没有蓝牙权限，返回默认值（海外用）<br/>
72、修复uid权限检查问题<br/>
71、安全性更新，内部应用的文件权限控制<br/>
70、提高内部app调用的兼容性，第三方登录，分享<br/>
69、自动过滤没权限的外部ContentProvider<br/>
68、增加功能：内部app的权限检查（默认关闭）<br/>
67、机型模拟:Build类和build.prop<br/>
66、提高对乐固加固的app兼容性<br/>
65、适配三星wifimanager<br/>
64、修复ipc框架一个参数传递问题（IPCMethod这个类必须更新）<br/>
63、补全7.0通知栏的hook<br/>
62、修正8.0动态快捷菜单的hook<br/>
61、SettingHandler新增一个适配接口，主要适配各种游戏<br/>
60、功能改动：google自动安装改为手动安装，避免第一次启动时间过久<br/>
59、可以禁止访问外部某个ContentProvider<br/>
58、适配华为桌面图标数量<br/>
57、权限分类注释，标注可删除权限。<br/>
56、增加双开模式的app跟随外部升级的开关。<br/>
55、提高app的jni兼容性。<br/>
54、提高对app集成其他插件框架的兼容性。<br/>
53、增加设置接口，根据包名进行设置。<br/>
52、增加Uri的适配范围，支持通过Uri分享和查看文件。<br/>
51、修复一个在三星8.0的问题。<br/>
50、提高对系统自带的app组件兼容性，更好兼容chrome webview，google service。<br/>
49、提高ART稳定性<br/>
48、增加相机适配范围<br/>
47、支持内部App在8.0下的快捷方式管理<br/>
46、修复exec异常<br/>
45、提高稳定性（修复微信登录闪退）<br/>
44、解决微信数据库崩溃问题<br/>
43、修复部分4.4设备崩溃问题<br/>
42、修复后台应用易被杀死，土豆视频黑屏，新浪微博无法打开，优酷两次返回无法退出。<br/>
41、增加应用的保活机制，双开APP更不易被杀死。<br/>
40、优化虚拟引擎启动性能。<br/>
39、兼容了大部分的加固，第三方APP兼容性对比上一版提升40%+。<br/>
38、修复某些rom下，快捷方式图标不正确<br/>
37、兼容以前组件StubFileProvider<br/>
36、适配部分新ROM的虚拟IMEI<br/>
35、改善进程初始化代码，增加稳定性<br/>
34、添加内部发送Intent.ACTION_BOOT_COMPLETED的广播，可以设置开关<br/>
33、适配关联google play游戏，支持游戏使用google登录<br/>
32、适配android O的google service框架<br/>
31、适配android O 快捷方式<br/>
30、适配耳机模式<br/>
29、某些rom对intent的大小限制，demo添加缩放快捷方式图标代码<br/>
28、修复多开情况下一个bug<br/>
27、修复某些情况下MediaController的bug<br/>
26、修复4.1.2的StubFileProvider报错<br/>
25、分享的uri处理<br/>
24、修复跨app调用Activity的回调<br/>
23、前台服务的通知栏拦截开关<br/>
22、附带doc<br/>
21、完善VA内部的intent的CHOOSE回调<br/>
20、Android O的通知栏适配2<br/>
19、ipc框架优化, 提高判断binder的存活准确性<br/>
18、jni的log开关 Android.mk:LOCAL_CFLAGS += -DLOG_ENABLE<br/>
17、混淆配置<br/>
16、Android O的通知栏适配<br/>
15、修复部分app网络卡的问题<br/>
14、适配 android 8.0的dl_open（jni加载）<br/>
13、修复华为emui8.0的一个bug<br/>
12、完善定位<br/>
11、设置手机信息，imei伪装算法<br/>
10、适配8.0某个功能（主要app：whatsapp）<br/>
9、修复内部微信等应用，无法更新图片，视频<br/>
8、demo增加安装监听，自动升级克隆模式的应用<br/>
7、7.0的file provider适配<br/>
6、增加了定位代码<br/>
5、代码进行了架构优化<br/>
4、与开源版不同的特征<br/>
3、解决了微信被封的一些问题<br/>
2、修复了部分机型兼容性<br/>
1、修复了12个小BUG<br/>
</details>






