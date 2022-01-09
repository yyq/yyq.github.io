---
layout: post
title: "apple iOS MobileHIG 摘要"
date: 2013-01-18 17:01:01
comments: true
tags: [apple,iOS,HIG]
---
一直想开发一个自己的app放在app store上，想先做一个有关数独的app。一个是因为我身边有不少人都喜欢玩这个游戏，还有一个是因为这个东西算法不是很难，之前用java和c#在android和windowsphone平台实现过。

等我idp申请下来，开始能上机调试，自己把数独的基本功能都搞定了之后，发现问题来了。我这程序吧，能够正常运行，仅仅能用。但是发现这界面啊，颜色啊，操作方式啊，都TMD烂透了。就开始琢磨这个事儿，这么样的程序提交app store铁定不会通过，自己已经为程序设计好一个图标，让同学和女友都看过，还过得去。但是点开图标之后的程序，那个实在是，我不知道该怎么下手来改进了。遂想起apple好像有个什么指南，告诉你移动开发者如何设计程序的人机交互的。在网上找了找，终于找到了这个资料,官网地址：[iOS Human Interface Guidelines](http://developer.apple.com/library/ios/#documentation/UserExperience/Conceptual/MobileHIG/Introduction/Introduction.html)

此文用来记录我从该文档中摘抄的我觉得现阶段对我有用处的东西。
<!--more-->
# Platform Characteristics
 * When people return to a suspended app, it can instantly resume running from the point where it went to the background, without having to reload its UI.
 * Two Types of Software Run in iOS
 	* iOS apps:is an app you develop using the iOS SDK to run natively on iOS device.
 	* Web content:is hosted by a website that people visit using their iOS devices.
 		* Web apps
 		* Optimized webpages
 		* Compatible webpages
 * 当人们从睡眠状态的app启动时，能马上恢复，从哪个状态进入后台的，要能立马重现那个状态，不需要重新载入界面。
 * iOS设备上有两类app
 	* 一类是本地app，只通过iOS SDK在本地运行在ios设备就ok
 	* 一类是网络app，这类程序又可以有三种
 		* 纯网页浏览，隐藏safari的按钮界面什么的，用户在操作的是网页上的元素
 		* 优化过的网页：专门为iOS优化过的网页，例如不用flash插件等，网页上的图片能根据ios设备的不同而进行不同的缩放
 		* 兼容并存的网页：和ios的safari兼容的网页和操作。

# Human Interface Principles
 * Direct Manipulation
 	* Rotate or otherwise move the device to affect onscreen objects
 	* Use gestures to manipulate onscreen objects
 	* Can see that their actions have immediate,visible results
 * Feedback
 	* Feedback acknowledges people's actions and asssures them that processing is occurring.
 	* The built-in iOS apps respond to every user action with some perceptible change.
 	* Subtle animation can give people meaningful feedback that helps clarify the results of their action.
 	* Sound can also give people useful feedback
 * User Control
 	* The best apps find the correct balance between giving people the capabilities they need while helping them avoid dangerous outcomes.
 * 直接操作，能让人们直接的动作在ios设备上得到反馈或者响应，如下几种典型的操作：
 	* 旋转或者移动设备，来影响屏幕上的物体
 	* 用手势来直接操作屏幕上的物体
 	* 可以看到我们的操作有直接的可视化的结果
 * 反馈
 	* 反馈可以ack人们的操作和告诉人们某个处理过程is doing now
 	* 内置iOS app对于每一个用户操作都会有一个某种类型的可感觉的反馈
 	* 一些微秒的动画，可以给人们有意义的反馈，来让人们更加明确他们操作的结果
 	* 声音的反馈也很重要
 * 用户控制
 	* 那些很好的app找到了一个很好的平衡点，来告诉人们该做什么操作，从而避免出现错误的结果

# App Design Strategies
## Create an App Definition Statement
 1. List All the Features You Think Users Might Like
 1. Determine Who Your Users Are
 1. Filter the Feature List Through the Audience Definition
 	* Now you can craft your app definition statement, concretely summarizing what the app does and for whom.
 1. Dont stop there
 	* As you consider whether to add a new feature,ask yourself whether it is essential to the main purpose of your app and to your target audience.
 	* As you consider the look and behavior of the UI,ask yourself whether your users appreciate a simple,streamlined style or a more overtly thmeatic style.
 	* As you consider the terminology to use,strive to match your audience's expertise with the subject.

## Design the App for the Device
make sure that your app looks and feels like it was designed expressly for an iOS device
### Embrace iOS UI Paradigms
 * controls should look tappable
 * app structure should be clean and easy to navigate
 * user feedback should be subtle,but clear
### Reconsider Web-Based Designs
if you're coming from the web,you need to make sure that you give people an iOS app experience,not a web experience.
 * Focus your app
 * Make sure your app lets people do something
 * design for touch
 * let people scroll
 * relocate the homepage icon

## Tailor Customization to the Task
 * be sure to consider customization early in the design process
 * you need to determine how customization might affect the task your app enables.As you consider the tasks in your app,think about how ofter users perform them and under waht circumstances.
 	* For example, imagine an app that enables phone calls. Now imagine that instead of a keypad, the app displays a beautiful, realistic rotary dial. The dial is meticulously rendered, so users both appreciate its quality and instantly know how to use it. The dial behaves realistically, so users delight in making the dialing gesture and hearing the distinctive sounds. But for users who often need to call numbers that are not in Contacts, initial appreciation of the experience soon gives way to frustration, because using a rotary dial is much less efficient than using a keypad. In an app that is designed to help people make phone calls, this beautiful custom UI is a hindrance.
 	* In this case,the custom UI not only shows people how to use the app,it also makes the task easire to accomplish.

 * Always have a reason for customization**
 * As much as possible, avoid increasing the user’s cognitive burden
 * Be internally consistent
 * Think twice before you redesign a standard control
 * Be sure to thoroughly user-test custom UI elements

# Transition Case Studies
 * 这一段的大部分内容主要就说的是如何从desktop app转换到ios app要注意的一些方法，两者的区别等等。我就不怎么关注了。只摘抄了一句话：
 * Most iOS apps don’t need to provide all the features that only power users need.
 
# User Experience Guidelines

### Focus on the Promary Task
### Elevate the Content that People Care About
### Give People a Logical Path to Follow
 * Make the path through the information you present logical and easy for users to predict.
 * In most cases,give users only one path to a screen.

### Make Usage Easy and Obvious
 * Make the main function of your app immediately apparent.
 * Be consistent with the usage paradigms of the built-in apps.

### Minimize the Effort Required for User Input
 * Make it easy for users to make choices. For example.you can use a table view or a picker instead of a text field, because it's usually easier for people to select an item from a list than to type words

### Downplay File-Handling Operations
 * although iOS apps can allow people to create and manipulate files,this does not mean that people should have an awareness of a file system on an iOS device.
 * As much as possible, allow people to manage documents without opening iTunes on their computer.

### Enable Collaboration and Connectedness
 * When appropriate, make it easy for people to interact with others and share things like their location, opinions, and high scores.

### De-emphasize Settings
 * Avoid including settings in your app if you can.
 * Let users set the behavior they want by using configuration options in your app. Configuration options let your app react dynamically to changes, because people don’t have to leave your app to set them.
 	* In the main UI, put options that provide primary functionality or that people want to change frequently.
 	* In iPhone apps, you can put options that people don’t need to change frequently on the back of a view.

### Brand Appropriately
 * Incorporate a brand’s colors or images in a refined, unobtrusive way
 * Avoid taking space away from the content people care about.

### Make Search Quick and Rewarding
if you need to provide search in your app, follow these guidelines to ensure that it performs well.
 
 * Build indexes of your data so that you are always prepared for search.
 * Live-filter local data so that you can display results more quickly
 * When possible, also filter remote data while users type
 * Display a search bar above a list or the index in a list
 * Use a tab for search only in special circumstances.
 * If necessary, display placeholder content right away and partial results as they become available. 
 * Consider providing a scope bar if the data sorts naturally into different categories.
 
### Entice and Inform with a Well-Written Description
Your App Store description is a great opportunity to communicate with potential users. In addition to describing your app accurately and highlighting the qualities you think people might appreciate the most, follow these guidelines:

 * Be sure to correct all spelling, grammatical, and punctuation errors
 * Keep all-capital words to a minimum
 * Consider describing specific bug fixes

### Be Succinct
 * Think like a newspaper editor,and strive to convey information in a condensed,headline style.
 * Give controls short labels,or use well-understood symbols
 
### Use UI Elements Consistently
 * Avoid radically changing the appearance of a control that performs a standard action.
 
### Consider Adding Physicality and Realism
 * Example:Contacts on iPad,Voice Memos app on iPhone

### Delight People with Stunning Graphics
 * Consider replicating the look of high-quality or precious materials
 * Create high-resolution artwork.
 * Spend the time to design a beautiful,memorable app icon.
 
### Handle Orientation Changes
 * In all orientations,maintain focus on the primary content
 * Think twice before preventing your app from running in all orientations
 	* Launch your app in your supported orientation,regardless of the current device orientation
 	* Avoid displaying a UI element that tells people to rotate the device.
 	* Support both variants of an orientation.
 
### Make Targets Fingertip-Size
 * Give tappable elements in your app a target area of about 44X44 pointx
 * Use animation consistently throughout your app.

### Ask People to Save Only When Necessary
### Start Instantly
 * Display a lunch image
 * Avoid displaying an About window or a splash screen
 * Launch in the appropriate default orientation
 * Avoid asking people to supply setup information
 	* Focus your solution on the needs of 80  percent of your users
 	* Get as much information as possible from other sources.
 	* If you must ask for setup information,prompt people to enter it within your app
 * Delay a login requirement for as long as possible
 * When your app restarts,restore its state so that users can continue what they were doing.
 
### Always Be Prepared to Stop
 * Save user data as soon as possible and as often as reasonable
 * Save the current state when stopping.
 


# iOS Technology Usage Guidelines

### Routing
讲地图的起点和终点切换的那个控件，估计我短期内用不到。

### Passbook
好吧，作为个人开发者，短期也不会用这个东西。

### Social Media
 * Give users a convenient way to compose a post without leaving your app
 * When possible,avoid asking users to sign into a social media account
 * Consider using an activity view controller to helop users choose one of their social media accounts.
 
### iCloud
### In-App Purchase
### Game Center

### Multitasking
To support this experience,multitasking allows an app to enter a suspended state in the background when users switch away from it.When users switch back to the app,the app can resume quickly because it doesn't have to reload its UI.

 * At a high level, this means that all apps should:
 	* Handle interruptions or audio from other apps gracefully
 	* Stop and restart--that is,transition to and from the background--quickly and smoothly
 	* Behave responsibly when not in the foreground.
 * Be prepared for interruptions,and be ready to resume
 * Make sure your UI can handle the double-high status bar.
 * Be ready to pause activities that require people's attention or active participation
 * Ensure that your audio behaves appropriately.
 * Use local notifications aparingly,
 * When appropriate,finish user-initiated tasks in the background.
 
### Notification Center

### Printing

### iAd Rich Media Ads

### Location Services and Data Privacy

### Quick Look Document Preview

### Sound

#### Define the Audio Behavior of Your App
 * if necessary,you can adjust relative,independent volume levels to produce the best mix in your app's audio output
 * Ensure that your app can display the audio route picker,if appropriate
 * if your app produces only UI sound effects that are incidental to its functionality,use System Sound Services.
 * If sound plays an important role in your app,use Audio Session Services

### Manage Audio Interruptions

### Handle Media Remote Control Events,if Appropriate

### VoiceOver and Accessibility

### Edit Menu
在文本处理的时候可以用到

### Undo and Redo

### Keyboards and Input Views
In iOS 4.2 and later,you can use the standard keyboard click sound to provide audible feedback when people tap the custom controls in your input view.To learn how to enable this sound in your code,see the documentation for *playInputClick* in UIDevice Class Reference.

# iOS UI Element Usage Guidelines
If you create custom UI elements to replicate standard UI elements in all but appearance, you should instead consider using the appearance customization programming interfaces available in iOS 5 and later.

### Bars
The UIStatusBarStyle constant, described in UIApplication Class Reference , controls the status bar style. To specify the status bar style, you set a value in your Info.plist file. 

 * Don't create a custom status bar.
 * When appropriate,display the network activity indicator
 * On iphone,specify the color of the status bar

### Navigation Bar
 * Use the title of the current view as the title of the navigation bar
 * Make sure it's easy to read the text in the navigation bar
 * Consider putting a segmented control in a navigation bar at the top level of an app.
 * Make sure it's easy to read the text in the navigation bar
 * Consider putting a segmented control in a navigation bar at the top level of an app.
 * Avoid crowding a navigation bar with additional controls, even if there appears to be enough space.
 * Use system-provided buttons according to their documented meaning.
 * If appropriate, customize the appearance of a navigation bar to coordinate with the look of your app.
 * If appropriate,customize the appearance of navigation-bar contrls.
 * On iPhone, be prepared for the change in navigation bar height that occurs on device rotation.
 
### Toolbar
* Use a toolbar to give people a selection of frequently used commands that make sense in the current context.
* If appropriate, customize the appearance of a toolbar to coordinate with the overall look of your app. 
* Maintain a hit target area of at least 44 x 44 points for each toolbar item
* Use system-provided toolbar items according to their documented meaning. 
* If appropriate, customize the appearance of toolbar items. 
* Try to avoid mixing plain style (borderless) and bordered toolbar items in the same toolbar.

### Tab Bar
* Don’t use a tab bar to give users controls that act on elements in the current mode or screen. 
* In general, use a tab bar to organize information at the app level
* Don’t remove a tab when its function is unavailable. 
* On iPad, you might use a tab bar in a split view pane or a popover 
* Consider badging a tab bar icon to communicate unobtrusively.
* Use system-provided tab bar icons according to their documented meaning. 
* If necessary, provide a custom selection indicator image.

## Content Views

### Activity

### Collection View
To learn more about defining a collection view in your code, see Collection View Programming Guide for iOS.
 
 * Don’t use a collection view when a table view is a better choice. 
 * Make it easy for people to select an item.
 * Use caution if you make dynamic layout changes.

### Container View Controller

### Image View

### Map View

### Page View Controller

### Popover(iPad Only)

### Scroll View

### Split View(iPad Only)

### Table View
a table view presents data in a single-column list of multiple rows.

If you want to lay out your table rows in a nonstandard way, it’s better to create a custom table-‐cell style than to significantly alter a standard one. To learn how to create your own cells, see “Customizing Cells” in Table View Programming Guide for iOS .

### Text View
it  accepts and displays multiple lines of text

### Web View

### Alerts, Action Sheets, and Modal Views
 * When possible, use a sentence fragment. A short, informative statement is often easier to understand than
a complete sentence.
 * Be sure to test the appearance of your alert in both orientations.
 * **Generally, use a two-button alert.** A two-‐button alert is often the most useful, because it’s easiest for people to choose between two alternatives. A single button alert is rarely helpful because it informs without giving people any control over the situation. An alert that contains three or more buttons is significantly more complex than a two-‐button alert and should be avoided if possible.
 
 * On iPad, an action sheet is always displayed within a popover;
 * On both devices, use the red button color if a potentially destructive action can be performed. 
 
### Modal View
 * On iPad, don’t display a modal view on top of a popover.
 * On iPhone, coordinate the overall look of a modal view with the appearance of your app. 
 * On both devices, display a title that identifies the task, if appropriate:For example, the Messages app on iPhone presents a modal view when users want to compose a text message. 
 
## Controls

### Activity Indicator
An activity indicator shows that a task or process is progressing 
 
 * Don’t display a stationary activity indicator, because users associate this with a stalled process.
 * If appropriate, customize the size and color of an activity indicator

### Date Picker
Countdown timer. The countdown timer mode displays wheels for the hour and minute. You can specify the total duration of a countdown, up to a maximum of 23 hours and 59 minutes.

### Contack Add Button
A Contact Add button lets the user add an existing contact to a text field or other text-‐based view (shown here in the To field of Messages).

### Detail Disclosure Button

### Info Button

### Label

### Network Activity Indicator
A network activity indicator appears in the status bar and shows that network activity is occurring 

### Page Control
A page control indicates how many views are open and which one is currently visible.

### Picker

### Progress View

### Refresh Control
To learn more about defining a refresh control in your code, see UIRefreshControl Class Reference.(恩，这是个新东西，应该是iOS6才出现)

### Rounded Rectangle Button

### Scope Bar

### Search Bar

### Segmented Control

### Slider
If appropriate, customize the appearance of a slider. 

### Stepper

### Switch

### Text Field
A text field accepts a single line of user input.

 * Display the Clear button in the right end of a text field when appropriate. 
 * Display a hint in the text field if it helps users understand its purpose
 * Specify a keyboard type that’s appropriate for the type of content you expect users to enter. 
 
### System-Provided Buttons and Icons

 * Use the system-provided items correctly
 * Avoid designing a custom icon that looks too similar to a system-provided icon
 
### Standard Buttons for Use in Toolbars and Navigation Bars

### Standard Icons for Use in Tab Bars
恩，意思是这里有一堆专用的图标，自己不用去创造新图标了
### Standard Buttons for Use in Table Rows and Other UI Elements
同前面一条。

# Custom Icon and Image Creation Guidelines

### Tips for Designing Great Icons and Images

 * For the best results,enlist the help of a professional graphic designer.
 * Use universal imagery that people will easily recognize
 * Embrace simplicity
 * Use color and shadow judiciously to help the icon tell its story.
 * In general, avoid using “greek” text or wavy lines to suggest text
 * In general, create an idealized version of the subject rather than using a photo
 * Portray real substances accurately
 * Use transparency when it makes sense. 
 
### Tips for Creating Great Artwork for the Retina Display

 * Richer in texture.
 * More detailed
 * More realistic
 
 * Start by scaling up your original artwork and then redraw it.
 * Add detail and depth.
 * Consider adding blur for better results in effects such as engravings and drop shadows. 
 
### Tips for Creating Resizable Images
Tiling is less performant than stretching, but it's the only way to achieve a textured or patterned effect.

### App Icons
YoucanpreventiOSfromaddingtheshinetoyourappicon.Todothis,youneedtoaddthe UIPrerenderedIcon key to your app’s Info.plist file (to learn about this file, see “The Information Property List” in iOS App Programming Guide ).

* Ensure that your icon is eligible for the visual enhancements iOS provides.You should provide an image that:
	* Has 90° corners (it’s important to avoid cropping the corners of your icon—iOS does that for you when it applies the corner-‐rounding mask)
	* Does not include a drop shadow
	* Does not have any shine or gloss (unless you’ve chosen to prevent the addition of the reflective shine)
	* Does not use alpha transparency
* Create a large version of your app icon for display in the App Store
	* 512 X 512 pixels, 1024 X 1024 pixels
	* Be sure to name this version of your app icon iTunesArtwork and iTunesArtwork@2x, respectively. 

### Launch Images
* Generally, design a launch image that is identical to the first screen of the app.
	* Exceptions:
	* Text. The launch image is static, so any text you display in it will not be localized.
	* UI elements that might change.
* For iPhone and iPod touch launch images, include the status bar region.
* For iPad launch images, do not include the status bar region. 
* Examples：
	* Settings的那个软件的启动界面，好吧，我就不放在我博客里了，自己去apple官网看吧
	* 还有股票那个软件的
	
### Small Icons
You can name your icon anything you want as long as you use the CFBundleIcons key to declare the names

### Document Icons

### Web Clip Icons

* If necessary, include custom pressed or selected appearances for your icons. 

### Newsstand Icons


































