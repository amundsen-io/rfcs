- Feature Name: frontend-library-split
- Start Date: 2020-11-23
- RFC PR: [amundsen-io/rfcs#150(https://github.com/amundsen-io/rfcs/pull/15) (after opening the RFC PR, update this with a link to it and update the file name)
- Amundsen Issue: [amundsen-io/amundsen#0000](https://github.com/amundsen-io/amundsen/issues/0000) (leave this empty for now)

# Split of the frontendlibrary repository in two

## Summary

At this moment, our frontend repository contains a Flask application that acts as an API server and also serves a frontend application based in React.js. This means that there are two applications (Flask and Frontend), with two languages (Python and JavaScript), two configuration settings, two dependency management systems (pip and npm), two sets of owners (frontend and backend developers), and two main "responsibilities" (API server and User Interface application) baked into the same repository. This is a problem, as it makes the system complex, couples both systems and stifles the evolution of both applications.

We propose to split our current amundsenfrontend repository into a backend-only API server and a frontend-only application.

## Motivation

> Why are we doing this? What use cases does it support? What is the expected outcome?

The main benefits of this proposal are:

- Easier evolution of the applications
- Allow for multiple frontends
- Streamlined development

### Evolution of applications

Having two applications in different repositories will reduce the coupling, allowing us to evolve both apps independently, implementing the newest best practices on both. For example, this would make it easier to create a GraphQL based interface for our API, as well as adopting a full stack structure based in Next.js or a static site based in Gatsby in the frontend.

### Allow for multiple frontends

Extracting the API server into its own applications would allow us to create different frontends for the Amundsen application. Think about creating a mobile application, a desktop app or forking the current UI application to customize it deeply to the user's specifications.

It would also make it easier for other applications to query Amundsen, integrating its results in other applications.

### Streamlined Development

Splitting the frontend repository would make it easier to develop in both systems, as updates and issues on one application won't affect the other. For example, we just had an issue where we couldn't test a change in the frontend app because a dependency on the backend app was incorrect.

Reducing the complexity of the project will improve developer's focus and create a simpler, more straightforward development experience.

## Guide-level Explanation (aka Product Details)

For a final user of Amundsen, there shouldn't be any change. This is an architectural-level refactor.

## UI/UX-level Explanation

No changes should be seen in the UI level, but this change would open doors for new and exciting evolutions on performance for the UI and the API responses.

## Reference-level Explanation (aka Technical Details)

Technically, the high level changes would be:

- Create a new repository for the frontend, probably based on a ready-made application like Next.js
- Move our frontend application into this new system, updating the endpoints and wiring to use the current frontend implementation
- Remove the frontend application from our current frontend repository

## Drawbacks

Why we shouldn't do this?

- Load of work involved
- Migration costs for current users

## Alternatives

We consider this the best approach as it lines up with general best practices, decouples our system and adds flexibility while reducing complexity.

One alternative would be to do this same work while keeping both in the same repository. It won't bring all the benefits, but it will be a bit better than the current status.

Not doing this would further leave us behind the latests improvements on the field, will continue having a complex setup and onboarding, making it hard to contribute to and hard to extend our platform.

## Prior art

We have talked several times about this, never tried it. It has been a common move in all internal projects at Lyft though.

## Unresolved questions

We'll need to create a detailed step-by-step plan for this refactor.

## Future possibilities

For the backend:

- Easily open the backend to other services and applications

For the frontend:

- Update it to use a static site generator like Gatsby
- Update it to become a Next.js application, incorporating best practices and best performance
