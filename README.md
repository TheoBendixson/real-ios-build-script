# A From-Scratch iOS Build System For An Actual Shipped Game
This is an example of a complete iOS app build system, written 100% from scratch, self-contained in a bash shell script.

It's not a theoretical build script for some hobby project that has not shipped. I used this script (and still use it) to build and update the [iOS version of my first game, Mooselutions](https://apps.apple.com/us/app/mooselutions/id6477404960?mt=12), a real product you can spend money on at the App Store.

Like any real shipping anything, it's kinda hairy and not all that "clean," which is exactly the reason I am posting it here, as-is. I want to show you an example of something real, something I interact with daily, warts and all.

# Why do this when you can just use Xcode and Xcodebuild?
I worked as an iOS developer for nearly a decade, and the lack of transparency when using xcodebuild always frustrated me. I didn't like that I have to invoke this magical tool and then figure out how to get it to do what I want it to do with all these adjacent scripts and settings within Xcode itself.

I wanted a build system that's simple and self-contained, something I can follow from start to finish as you would any ordinary program.

I also wanted to make a build system that can easily do the unity build (not to be confused with the game engine) that you see in Casey Muratori's Handmade Hero. I'm a huge fan of the series, and it wasn't clear to me how I could reconcile Casey's build system with Apple's xcodebuild tool. So I decided to ditch xcodebuild.

The game itself (Mooselutions) is cross-platform (it also runs on Mac OS and Windows). It has some unique constraints you wouldn't find in the typical iOS app, so for that reason it probably shouldn't use the typical iOS app build system. 

I like how this has turned out, but it has its quirks. It's certainly not the most convenient build system to set up, but I think it's more maintainable in the long run.

I really love the transparency and ability to simply look at the line where something happens, reason about it, make a change, and get exactly the result I want. If we're a of a similar mindset, I think you'll love that too.

# What's happening in this script?
At a high level, the script draws from a resources and source code directory to compile the iOS version of my game, Mooselutions. Once the game is compiled into an executable, it creates the app bundle structure in exactly the way Apple wants it to be made for Xcode version 15.2, iOS version 17.0

Once the bundle is created, it signs the bundle using entitlements and a mobile provisioning profile. There are different options for different deployment targets. You can build for the iOS simulator or an iOS device, and once the build is done, you can go into Xcode to install the app on your phone and use the debugger if needed.

The script also has a release process that signs the app bundle with distribution certificate, and then creates an .xcarchive file folder which Xcode recognizes. You can then use Xcode to upload the build to the App Store, including dSYMs for crash reporting. 

# All the gotchas explained
It wouldn't be Apple if there weren't a ton of undocumented rules you've gotta follow. Here's an explanation of all the trouble I ran into while creating this.

## Xcode Versions
This build script only works for Xcode 15.2 with a minimum deployment target of iOS 17. As such, it has adopted all of the acceptable file structure conventions for that moment in time, and it does not support any other Xcode version.

I'm just one guy who needed to ship one game. I couldn't find a single example of the thing I wanted to do (for any version of Xcode), so I humbly offer the thing that solved my problem.

If you need to support a different version of Xcode, you will need to write a slighly different version of this script. The best way to do that is to download the version of Xcode you want to use, create a default vanilla iOS application, build it, and then inspect the bundle to see how everything is laid out. Deconstruct what Apple's tools make so you can then re-construct it with your own tools.

## App Icons
You will notice that there's a file called Assets.car in the resources directory. That's just the game's App Icon. In order to avoid using xcodebuild, I had to create a separate Xcode project to create the icon from an xcassets folder within Xcode. I built the sample app, went into the build bundle's content directory, and then copied Assets.car from it into my own resources directory.

I also copied the other icons (AppIcon60x60 for example) from that build directory into my own resources directory. My build script then copies them from my resources directory into my app bundle, and it also includes the approprite Info.plist keys to make sure iOS knows about the icons on both iPhone and iPad.

It's annoying and it's a thing you have to do outside of this build script (I guess I lied about it being self-contained), but I figure you only have to do it once for most projects since the App Icon doesn't change all that often.

## Main.storyboard and Launch.storyboard WTF?
I know. I think it's silly that you have to specify these in order to get a game running on iOS, but that's how it seems to be these days.

My Main.storyboard doesn't do anything other than specify the custom UIViewController subclass from the one screen in my game, and my Launch.storyboard is just a blank black screen.

I only use one UIViewController, and it has just one subview that renders Metal commands into it. The Main.storyboard hooks that up.

Trust me, I tried a bunch of different options and really wanted to avoid using any storyboards at all, but I couldn't find a way to do it in the time constraint, so this is what I've got.

## Don't mess around with the Info.plist
I learned the hard way that the Dictionary abstraction advertised with Info.plist is a total lie. If you change the ordering of some keys, it screws up the app submission process and you will get a ton of cryptic errors that will lead you on a wild goose chase. So just don't do that.

I shit you not, the *only* way I was able to submit this thing was to use Xcode to create a default iOS application, build it, and then copy the generated Info.plist line by line, verbatim.

It's probably not as strict as I'm making it out to be, but why take the chance? There are tools in Apple's submission pipeline that do not treat Info.plist as an NSDictionary. I'm pretty sure some of those tools read it linearly, so if a key occurs in the wrong order, you can get the tool to get out-of-step and very confused.

# What to do if you're stuck
Apple Guessing (TM) sucks. It's one of my least favorite things to do as a developer. There's some opaque system that's spitting out errors, but it won't tell you the internals of what it's doing or how it came to whatever incorrect conclusion it has hit upon.

If you find yourself in App Store Submission Hell, just slow down and Work The Problem. If you can think of something to try, try it. If you can't presently think of something to try, don't force it. Go take a shower. Go cook dinner. Take the dogs on a walk. Do anything else.

I found it was most helpful to use Xcode's default template projects as a reference and starting point. Build them, and then click the option to show them in the Finder. Inspect the bundle and compare it with your build system's results. If your build system didn't produce exactly the thing that Apple's default project produced, fix your build system so it does. 

Apple is very strict like that.

If you go searching on Google, chances are you're going to find a bunch of clueless people offering half-baked mentally absent non-solutions like "clean the build folder," or "I just deleted all of my provisioning profiles, re-downloaded them, and it worked."

Don't listen to any of the noise. By taking this approach and doing it from scratch, you automatically know far more about the internals of Xcode's build system than they ever will. This isn't about flicking a checkbox and saying "it worked." Take the time to actually understand why you're getting the error you see, fix it, and don't ever have the problem again.

Best of luck, and if you found this useful, [please join my mailing list](https://us14.list-manage.com/contact-form?u=c5044b427cbe00b3ddb3882bb&form_id=b722e08e282b4161610adc11783032b3). 

I regularly send updates showing off new things I've learned, games I'm working on, and general wisdom.



