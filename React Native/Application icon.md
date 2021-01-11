This chapter will lead you through the basics needed to add the application icons to your RN app.

### Android

 1. To add application icons first ask designer to provide you with
    icons in **.png** format and [sizes](https://makeappicon.com/androidicon).

Make sure that designer provides you both type of icons (square and round ones), because android added adaptive icons since version 8.0. Basicly, this allows android system to choose which icon type to display.

 2. If your application doesn't have multiple env. skip till step 6.
 3. Navigate to `android/app/src` directory and create separate folders for each of the environment (flavor) you have defined inside your gradle file. Directory name has to match the product flavor name.
 4. Each "flavor" has to contain the `res` directory with appropriate `mipmap-<size>` subdirectories and playstore icon.
 5. After you've set up this for all your flavors, you can delete `src/main/res` directory since it will be used only if icons are not found inside your flavor directories (basicly it's a fallback).
 6. Add your icons to their designated `mipmap` directories according to sizes above.

### iOS

1. Navigate to iOS directory and open your project with XCode using .xcworkspace file.
2. Inside XCode find **Images.xcassets** directory and select the  **AppIcon** file at top of the window you
3. Add all icons to their designed boxes as defined.

#### Multiple env. (option 1 - targets)
Add multiple icon sets (each target has its own icons) using **+** sign inside **Images.xcassets** window and select `App icons & Launch images -> New iOS App Icon`. Change the name of the icon file to reflect your target name: `AppIconStaging`.
On the right side of the icons window find `Target Membership` section and select related target. Make sure you do this for all icon sets.

#### Multiple env. (option 2 - configurations)

Define multiple icon sets as described in **option 1** and change their names so they are easily related to their configuration:
`AppIconDev`, `AppIconProd` etc.
After you've added all the related icons select the project target (you should have only 1) and under `Build Settings` navigate to `Asset Catalog Compiler - Options`. Expand `Asset Catalog App Icon Set Name` and change the name of the icons so they match ones from the **Images.xcassets** you defined previously.