# How to build adaptive UI with Flutter

🔗 [link to video](https://www.youtube.com/watch?v=LeKLGzpsz9I)
🔗 [Material3](https://m3.material.io/)
🔗 ["Building an animated responsive app layout with Material 3"](https://codelabs.developers.google.com/codelabs/flutter-animated-responsive-layout#5)

## SafeArea widget

SafeArea blocks off space on the top/bottom for device UI things, like cutouts, status bars, etc.

AppBar does this automatically, which is why is recommended to use it for top navigation

## GridArea

Don't set column count by the device type (Android vs iOS vs tablet vs desktop)

Instead, set it by the window size. Not the device size itself, but the amount of space the window is giving us

- Many iOS and Android devices support some sort of multi-window mode, which will cause your app to be rendered in a space smaller than what the physical size of the display is

```
GridView.builder(
    itemBuilder: (BuildContext context, int index) {
        return CustomImage(
            images[index].url,
            onTap: () {
                print('_onTap');
            },
        );
    },
    // this is the important part
    gridDelegate: SliverGridDelegateWithMaxCrossAxisExtent(
        maxCrossAxisExtent: 250
    )
)
```

The `250` is the max width of our items.

Layout guidance from [Material3](https://m3.material.io/) is here. For example, it's currently best practice to not let the screen content take up the entire screen on desktop. Leave some white space on either side.

## Foldables

DO NOT LOCK SCREEN ORIENTATION

Weird things can happen, especially with foldable phones

## Adaptive Widgets

Dialogs, Navigation UI, and Custom Layout

### MediaQuery

Lots of information about where your app is running

```
const MediaQueryData({
    this.size,
    this.devicePixelRatio,
    textScaleFactor,
    etc.
})
```

`this.size` gives you the size of the window itself

#### Do use:

`final windowSize = MediaQuery.sizeOf(context);`

- It will cause a re-build if the size property changes

#### DON'T use:

`final windowSize = MediaQuery.of(context).size;`

- It will cause a re-build if ANY of the properties inside of MediaQuery change, resulting in lots of unnecessary rebuilding and wasted resources

### LayoutBuilder

- Helps us accomplish a similar goal as MediaQuery.sizeOf, but with some important distinctions
- Rather than always giving us the size of the window, it gives us the layout constraints from the parent widget
  - It gives us the min/max height and width allowed for our widget
- This means that you'll get sizing information based on the specific spot in the widget tree that you add the LayoutBuilder to

### Dialogs

- Let's say you have content that you want to be a full-screen on mobile and a modal on larger screens.
- Flutter's dialog class has two helpful constructors. The default will create the modal, and using `Dialog.fullscreen()` that will take up the entire screen.
- But how do we use both? Do we really have to duplicate all of our code?
- Nope! Both constructors have a `child` parameter so you can reuse the same child widget for both.

- Dialogs are good things to use `const` with, especially if they're stateless, because it lets Flutter know that it doesn't need to reconstruct this widget instance during rebuilds

```
showDialog(context: context, builder: (context) {
    const dialogContent = DialogDetailScreen(...);
    final showFullScreenDialog = MediaQuery.sizeOf(context).width < 600;
    if (showFullScreenDialog) {
        return const Dialog.fullscreen(
            child: dialogContent,
        );
    } else {
        return const Dialog(
            child: ConstrainedBox(
                constraints: BoxConstraints(maxWidth: 400),
                child: dialogContent,
            ),
        );
    }
})
```

1. Abstract -- find shared data and abstract it
2. Measure -- choose between `MediaQuery.sizeOf` or `LayoutBuilder`, depending if we want the size of the whole app window or more local sizing
3. Branch -- add breakpoint logic

### Navigation UI

We want a nav bar that is on the bottom on mobile and a 'rail'(?) on larger.

#### Step 1: Abstract -> Create shared destinations

```
class Destination {
    const Destination(this.icon, this.label);
    final IconData icon;
    final String label;
}

const List<Destination> destinations = <Destination>[
    Destination(Icons.image, 'Quick Caption'),
    Destination(Icons.upload, 'Bulk Upload'),
];
```

```
class BottomNavBar extends StatelessWidget {
    final int selectedIndex;
    final ValueChanged<int>? onDestinationSelected;

    const BottomNavBar({
        super.key,
        required this.selectedIndex,
        this.onDestinationSelected,
    });

    @override
    Widget build(BuildContext context) {
        return NavigationBar(
            selectedIndex: selectedIndex,
            onDestinationSelected: onDestinationSelected,
            destinations: destinations.map((dest) {
                return NavigationDestination(
                    icon: Icon(dest.icon),
                    label: dest.label,
                );
            }).toList(),
            ...
        );
    }
}
```

```
class SideNavRail extends StatelessWidget {
    final int selectedIndex;
    final ValueChanged<int>? onDestinationSelected;

    const SideNavRail({
        super.key,
        required this.selectedIndex,
        this.onDestinationSelected,
    });

    @override
    Widget build(BuildContext context) {
         return NavigationRail(
            selectedIndex: selectedIndex,
            onDestinationSelected: onDestinationSelected,
            destinations: destinations.map((dest) {
                return NavigationRailDestination(
                    icon: Icon(dest.icon),
                    label: dest.label,
                );
            }).toList(),
            ...
        );
    }
}
```

#### Step 2: Measure -> What data should drive the layout decision?

Because we want the layout to change based on the size of the whole app window, we'll use `MediaQuery`.

```
class PhotoAppRoot extends StatefulWidget {
    ...

    @override
    Widget build(BuildContext context) {
        final useSideNavRail = MediaQuery.sizeOf(context).width >= ???;
        return Scaffold(
            appBar: AppBar(...),
            body: BodyWidget(...),
            ...
        )
    }
}
```

At what size do we want `useSideNavRail` to be true??

#### Step 3: Branch -> At what breakpoint should we use a `NavigationRail` instead of a `NavigationBar`?

M3 suggests that you use a NavBar when the size is less than 600px and a NavRail for those > 600. Let's start there.

```
class PhotoAppRoot extends StatefulWidget {
    ...

    @override
    Widget build(BuildContext context) {
        final useSideNavRail = MediaQuery.sizeOf(context).width >= 600;
        return Scaffold(
            appBar: AppBar(...),
            body: BodyWidget(...),
            ...
        )
    }
}
```

Now let's create the branching logic.

```
class PhotoAppRoot extends StatefulWidget {
    ...

    @override
    Widget build(BuildContext context) {
        final useSideNavRail = MediaQuery.sizeOf(context).width >= 600;
        return Scaffold(
            appBar: AppBar(...),
            body: Row(
                children: [
                    if (useSideNavRail) SideNavRail(...),
                    Expanded(child: BodyWidget(...)),
                ],
            ),
            bottomNavigationBar: useSideNavRail ? null : BottomNavBar(...)
            ...
        )
    }
}
```

Check out the ["Building an animated responsive app layout with Material 3"](https://codelabs.developers.google.com/codelabs/flutter-animated-responsive-layout#5) codelab for lots more information and detail

### Custom Layout

We'll build a container with certain image details in a row if there's space, or vertically if not.

#### Step 1: Abstract

```
class AdaptiveDetailsContent extends StatelessWidget {
    final Widget image;
    final Widget caption;
    final Widget description;
    final Widget tags;
}

const AdaptiveDetailsContent({
    super.key,
    required this.image,
    required this.caption,
    required this.description,
    required this.tags,
});

@override
Widget build(BuildContext context) {
    return const Placeholder();
}
```

#### Step 2: Measure

For this widget, we want the sizing to be based on whatever space is given to specifically our widget, not the app window in general.

```
class AdaptiveDetailsContent extends StatelessWidget {
    final Widget image;
    final Widget caption;
    final Widget description;
    final Widget tags;
}

const AdaptiveDetailsContent({...});

@override
Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
        return const Placeholder();
    });
}
```

#### Step 3: Branch

When do we want to switch between the layouts?

Based on a rough sketch of what we want it to look like, we could set the horizontal layout when the size given to the widget is longer than it is tall and vice versa.

```
@override
Widget build(BuildContext context) {
    return LayoutBuilder(builder: (context, constraints) {
        final isMoreTallThanWide = constraints.maxHeight > constraints.maxWidth;
        if (isMoreTallThanWide) {
            return Column(
                children: [image, caption, description, tags],
            );
        } else {
            return Row(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                    image,
                    Expanded(
                        child: Column(
                            crossAxisAlignment: CrossAxisAlignment.start,
                            children: [caption, description, tags],
                        ),
                    )
                ],
            );
        }
    });
}
```

## Adaptive Inputs

set things like focused states and stuff, use animations, to show what is selected on all screen sizes. Think about things like stylus input or D-pad.

## Things to keep in mind

Break down your widgets!!!! Keep things easily digestible (and constant)

Having small, constant widgets can result in faster build times vs larger, complex widgets

Breaking your layout into Compact, Medium, and Expanded layouts is a good place to start. Look at the m3 guidelines for more info

- Compact -- less than 600px wide
- Medium -- between 600 and 840
- Expanded -- above 840
