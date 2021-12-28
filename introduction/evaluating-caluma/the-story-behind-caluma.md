# The story behind Caluma

If you have experience with the development of business applications, you might have found that many apps have a lot in common: Often they are trying to bring an existing business process into the digital world by implementing some **standard** functionality (e.g. forms, workflows, document mangagement and -generation, authentication, user management, analytics) and some more specific, **domain-specific** functionality (e.g. interface to custom third-party systems, special UI components, ...).

But even if two applications share a lot of standard functionality, their implementations often have not much more in common than maybe the general-purpose framework and some lower-level libraries (if at all). Why is that? There are probably countless reasons for this - one important one being the **cost of abstraction**: Standard, "off-the-shelf" tools need to be customizable so they can be applied to different use-cases. This requires abstractions, which necessarily means that the standard tool is more complex than what your specific use-case might require. This complexity can make the tool harder to learn, harder to modify, and often also harder to operate. If the abstractions don't match well with your requirements, it might mean that instead of benefiting from the existing functionality, you find yourself fighting it.

The team behind Caluma has been developing tailor-made solutions for many years. The resulting solutions typically are not more complex than they need to be, and relatively easy to modify. However, the maintenance effort usually has to be carried by the project alone as well. If a specific application has been in "maintenance mode" without significant feature development for many years, it can mean that the underlying software stack becomes outdated and/or the necessary know-how to work with the application is not available anymore. This can make changes to the software more expensive and ultimately require an even more expensive re-build from scratch.

Learning from the experience with a wide range of projects, with Caluma we found an abstraction where the benefits outweigh the downsides for a specific class of use-cases: Applications that implement a business process that revolves around forms. How this abstraction looks like will be covered in the following chapter.

After using Caluma in several projects across different industries, we've found the approach to have several advantages:

* It allows us to focus on the interesting, domain specific requirements of the customer while being able to re-use most of the recurring, standard functionality.
* Since Caluma is an Open Source project, improvements benefit the entire community, and the maintenance costs can be shared.
* New team members find it easier to become productive in a new (Caluma-based) codebase, because they share a common foundation.
