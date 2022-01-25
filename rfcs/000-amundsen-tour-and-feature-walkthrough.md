- Feature Name: Product Tour
- Start Date: 2020-01-25
- RFC PR: [amundsen-io/rfcs#0000](https://github.com/amundsen-io/rfcs/pull/0000) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Amundsen Product Tour and Feature Walkthrough

## Summary

The Product Tour for Amundsen will be a UI based walkthrough configurable component that will help onboard users into Amundsen. Alternatively, it will help us promote new features added to Amundsen, and educate our users about its use.

## Motivation

Currently, Amundsen has "help tips" icons with tooltips that explain some features. We also have an announcements block in our homepage where we show the latest additions to the product. These can be useful, but they are sometimes inssuficient to help our users to get onboarded or discover new features. We think we can create a better experience to our new users and a quick up to date for existing ones.

The outcome of this feature will be an easier onboarding of new users and an ad-hoc introduction to new features as we add them to Amundsen.

## Guide-level Explanation and UI/UX-level Explanation

For Amundsen users, the Tour will trigger in two different modes. The first is a general "Getting started with Amundsen" walkthough, while the second will highlight different features. Both would be formed by an overlay and a modal that will be attached to elements in the UI.

This modal window will have a "Dimiss" button that would hide the Tour altogether; a "Back" button that would move the user to the previous tour step, a "Next" button that moves it forward and a "Close" button with the usual "X" shape in the top right corner. You can see here [an example](https://react-joyride.com/) of what this tour can do (click on the "Start" button).

For Amundsen maintainers, we will extend the JavaScript configuration file with a block about the tour. This object will have a shape like this:

```js
...,
tour: [
    {
        path: '/',
        isFeatureTour: false,
        isShownOnFirstVisit: true,
        isShownProgramatically: true,
        steps: [
            {
                target: '.logo',
                title: 'Welcome to Amundsen',
                content: 'Hi!, welcome to Amundsen, your data discovery and catalog product!',
                ... // More options from those listed on https://docs.react-joyride.com/step
            },
            {
                target: 'h1.bookmarks-section-title',
                title: 'Save your bookmarks',
                content: 'Here you will see a list of the resources you have bookmarked',
                ... // More options from those listed on https://docs.react-joyride.com/step
            },
            ...
        ]
    },
    {
        path: '/',
        isFeatureTour: true,
        isShownOnFirstVisit: true,
        isShownProgramatically: true,
        steps: [
            {
                target: '.search-input',
                title: 'Search behavior updates',
                content: 'We have improved our search! Now you can do fuzzy search and this and that',
                ... // More options from those listed on https://docs.react-joyride.com/step
            },
            ...
        ]
    }
],
..
```

## Reference-level Explanation (aka Technical Details)

We will base this feature on [React Joyride](https://docs.react-joyride.com/), a third party React component that we are already using at Lyft at the moment.

We will extend its regular features with a persistance layer based on localStorage to allows us to show the tour only on the first visit.

## Drawbacks

- It could be tricky to cover ALL the use cases, but we think this will be straightforward and better than nothing.
- It won't be a breaking change, only installations with the mentioned configuration will trigger the Tour.

## Alternatives

Not doing this would me to keep on depending on the Announcements section to update users with new features. It will also mean that the onboarding of new users will need to be backed by documentation, instead of having it in the app itself.

## Prior art

We have seen this kind of feature working well in other applications of all kind of type.

## Unresolved questions

We don't consider using a different third party, as we have most of the work already figured out and working on a different Lyft application. That said, if once we launch this feature somebody in the community wants to improve it, they will have our approval.

## Future possibilities

Haven't though much about this.
