+++
title = 'Lottie Animation in Flutter'
date = 2024-05-05T21:21:23-07:00
+++

There are different approaches of creating animations in Flutter. The Flutter documentation [Introduction to animations](https://docs.flutter.dev/ui/animations) outlines a decision tree in choosing the approach of animations in Flutter.

My use case is relatively straightforward - a confetti effect when a modal opens. Before thinking about custom painter, I explored the options of third-party Flutter packages and was happy with the result of the lottie package, which parses animations exported as JSON and renders the animations on mobile.

## LottieFiles

[LottieFiles](https://lottiefiles.com/) is a platform that lets you create and share a Lottie file, which is essentially a JSON-based animation format.

For example, following is a screen recording of the Lottie animation I intended to use in my Flutter project:

{{< figure src="/lottie.gif" title="Lottie Animation" width="50%" >}}

To use the Lottie animation, export the animation as JSON.

## Lottie for Flutter

The lottie package loads the JSON asset and renders the animation. By default, the lottie package would run the animation indefinitely. To configure the animation, use a custom `AnimationController` to control the animation effect. Here's an example to load the confetti animation:

```
class ConfettiWidget extends StatefulWidget {
  const ConfettiWidget({super.key});

  @override
  State<ConfettiWidget> createState() => _ConfettiWidgetState();
}

class _ConfettiWidgetState extends State<_ConfettiWidgetState>
    with TickerProviderStateMixin {
  late final AnimationController _controller;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(vsync: this);
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return IgnorePointer(
        ignoring: true,
        child: Lottie.asset('assets/confetti.json', controller: _controller,
            onLoaded: (composition) {
          _controller
            ..duration = composition.duration
            ..forward();
        }));
  }
}
```

Here's an example of the animation in Flutter app:
{{< figure src="/app_recording.gif" title="Animation in Flutter" width="50%" >}}
