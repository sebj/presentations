# MotionLayout

MotionLayout is something that was introduced at Google I/O 2018 in May, as part of the ConstraintLayout 2.0 support library. It’s still in alpha, but stable enough to be worth a look.

So, it’s a new layout to animate views and transition between layouts. More importantly, it’s a subclass of ConstraintLayout, which means that transitions are driven by constraint sets (which we’ll talk more about shortly).

And finally, it’s fully declarative – transitions between states can be fully described in `XML`. No code is expected, nor is it possible!

So, taking a high-level view, what does this look like?

Your layout file is defined as usual – the only requirements are that it uses a `MotionLayout` as the root tag, instead of a ConstraintLayout, and that all views have IDs.

What the `MotionLayout` actually does, is defined in a separate XML file, a `MotionScene`, that’s stored in the res/xml directory.

This file defines the layout states, in terms of `ConstraintSet`s. For example, a start constraint set defining a view on the left of the screen, and second constraint set defining that same view on the right. Each constraint set can change both position constraints, and almost any other attribute, including custom attributes, and can change multiple views at once.

The file also defines the transitions to use between those `ConstraintSet`s and any interactive triggers, like `OnTouch` to transition when the layout is tapped, or `OnSwipe`, to transition when the layout is swiped in a particular direction. It’s very flexible – a tap can be set to transition to the start, end, toggle or jump to one end.

## Animating a Square

So in practice, what does this look like? I just want to talk through an example that might be a bit contrived, but should illustrate the point… If we had a `ConstraintLayout` with a view we wanted to animate, say a square in an uninspiring view… We could change from a `ConstraintLayout` to a `MotionLayout`, and set the `layoutDescription` attribute, which will point to our motion scene `XML` file.

Now the important part: our `MotionScene` `XML` file. We’d like to animate a view from one state, to another, so we define two constraint sets. A start, and an end. We’ll centre the view horizontally… …and then have it start at the bottom, and end at the top.

Defining a transition between these constraint sets is done by creating a `Transition` tag, setting the start and end attributes to point to those constraint sets, and setting a duration.

Now at the moment, there’s nothing to trigger this transition.

The `setTransition` method could be pretty powerful, depending on the use case, to dynamically change transitions at any point. You could change either of the start and end constraint sets, while leaving any XML-based triggers intact (we’ll get onto those in a minute).

But, going back to the declarative side of `MotionLayout`, the power comes with being able to easily use click or swipe interactions to not only trigger transitions, but to be able to seek to any position in the transition, so you can drag your finger halfway up the screen to transition halfway, and when you let go it’ll carry on animating upwards, calculating the duration and interpolation for you.

## Recreating `BottomSheet`

As a small test, I tried to see if I could recreate a `BottomSheet`-like behaviour with only `MotionLayout`. Here we've got a button to toggle the sheet, a dark overlay view covering the layout and the actual bottom sheet layout itself inside.

The MotionScene is a little lengthy. There’s an `OnSwipe` method to handle dragging of the bottom sheet, and an `OnClick` event on the button. As for the constraint sets, we start with an invisible overlay and the sheet “underneath” the screen, and end with the overlay slightly visible to darken the screen, and the bottom sheet animated upwards and visible.

You could choose to add a third constraint set, and trigger animations between closed, half closed and open states set in code perhaps.

## Wrapping up

[Chris Banes' animated example for his TV show tracking app Tivi, on Twitter](https://twitter.com/chrisbanes/status/1029619278863945728)

---

At Google I/O earlier this year, a motion editor built into Android Studio was demonstrated. They showed a timeline that can be scrubbed through, with adjustable keyframe markers and a panel to customise attributes for transitions, interactions and views themselves.

Unfortunately, we’ve yet to see any of this, even in Android Studio Canary!

> While we are actively working on this tool, it’s not available yet. It will likely be available once the library reaches stable / beta.

[(*Introduction to Motion Layout (Part I), Google Developers*)](https://medium.com/google-developers/introduction-to-motionlayout-part-i-29208674b10d)

---

To round off this talk, I thought I’d give a mention to two talks that are definitely worth taking a look at if you haven’t already. First, [What’s New with ConstraintLayout and Android Studio Design Tools from Google I/O in May](https://www.youtube.com/watch?v=ytZteMo4ETk&feature=youtu.be&t=28m45s). I’ve referred to it a few times, but it’s a great source of information not just on `MotionLayout` but on the many changes around `ConstraintLayout` 2.0, like the linear constraint layout and flow constraint layouts, and the new constraint helpers.

A session from Apple’s WWDC conference I’d recommend is [Designing Fluid Interfaces](https://developer.apple.com/videos/play/wwdc2018/803/). This is a bit of a curveball for an Android-focused presentation, but it’s a very interesting look at interaction, UI and animation design, and covers a few practical tips for dealing with user interaction that are applicable cross-platform, like tracking acceleration and velocity instead of just the position of gestures, and amplifying that movement outward.