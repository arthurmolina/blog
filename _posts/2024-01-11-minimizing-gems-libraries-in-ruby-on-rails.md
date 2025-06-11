---
title: "Minimizing Gems/Libraries in Ruby on Rails: Keeping Code Within the Repository"
lang: en
last_modified_at: 2024-01-11T16:00:00-03:00
categories:
  - Blog
tags:
  - code
  - electronics
  - arduino
show_overlay_excerpt: false
toc: true
header:
  image: /assets/images/nonsense.jpg
  overlay_image: /assets/images/nonsense.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

When we are developing web systems it is easy to add a new gem or library without thinking much about it. It does what we need, if I'm going to have to create that functionality, I might need a long time. It will save me a lot of work. The code is already there ready to be used. It has an entire documentation, tests and (sometimes) a whole community providing support and developing new features.

So… Why not?

However, while gems and libraries can offer many benefits, there are compelling reasons to exercise caution and minimize their usage in favor of keeping code within the repository.

# Version Incompatibilities

One of the foremost concerns when relying heavily on external gems and libraries is the risk of version incompatibilities. As Rails and its associated gems evolve, newer versions may introduce breaking changes or conflicts with existing code. This can lead to painstaking troubleshooting, dependency resolution, and potential downtime. By minimizing external dependencies, developers can mitigate the risk of version conflicts and maintain greater control over their codebase's stability.

I ran into a problem like this recently: I was working on an application that was many years old with an outdated version of Rails and Ruby. Many gems were old and needed to be updated. What happened was that one of the gems needed to be updated anyway because it was integrated with a third party service. To be able to carry out the update, it was necessary to update several other gems together, causing uncertainty as to whether everything would work out. Good thing we had acceptable test coverage and a wonderful QA team.

# Code Maintenance

Every gem or library added to a Rails project represents another piece of code that must be monitored, updated, and maintained. As time passes, maintaining compatibility with evolving dependencies becomes increasingly challenging, particularly if certain gems fall out of active development or lack adequate documentation. By keeping code within the repository, developers can simplify maintenance efforts, ensuring that all components of the application remain up-to-date and cohesive.

In the same code mentioned above we have several challenges that for now are just technical debts but that at an inopportune moment we will have to deal with. For example, this application is using a gem that is already deprecated, when we need to update another gem that it depends on, we will need to modify all the code where it is used to place a replacement. That time will come, one way or another.

# Security Risks

External dependencies can introduce security vulnerabilities into an application. While reputable gems are typically maintained with security best practices in mind, vulnerabilities may still arise, especially in lesser-known or unmaintained libraries. By minimizing the number of external dependencies, developers reduce the attack surface and can more effectively implement security measures tailored to their specific application.

At the same time, we are targets of hackers who can deliberately add backdoors and malicious codes to the packages downloaded for our applications. The media has already recorded several cases of security breaches in libraries, mainly from [NPM](https://www.securityweek.com/dozens-of-malicious-npm-packages-steal-user-system-data/). But RubyGems is also not safe from [malicious](https://www.securityweek.com/two-malware-laced-gems-found-rubygems-repository/) [code](https://www.theregister.com/2019/08/20/ruby_gem_hacked/) insertion.

# Performance Overhead and Dependency Bloat

Every gem/library included in a Rails project incurs a performance overhead, however minor. While individual performance impacts may seem negligible, they can accumulate over time, particularly in large-scale applications with numerous dependencies. Minimizing external dependencies allows developers to optimize performance by reducing unnecessary overhead and streamlining the application's execution.

Over-reliance on gems/libraries can lead to dependency bloat, wherein a Rails project becomes burdened with an excessive number of external dependencies. This can impede development agility, increase deployment complexity, and hinder scalability. By prioritizing self-contained code within the repository, developers can maintain a leaner, more manageable codebase that is easier to maintain and scale over time.

# Practices

The author of the book [Sustainable Web Development with Ruby on Rails](https://sustainable-rails.com/), David Copeland, suggests that we should update dependencies Early and Often. His advice is to define one day per month for dependencies to be updated. In other words: “we’d run bundle update in our Rails apps, run the tests, fix what was broken, and then be up to date” (page 380). Another piece of advice from the author is to keep a Versioning Policy and Automate Dependecy Updates.

Another good way to be prepared for package updates, in addition to using minimal libraries, is to have good automated test coverage. And never forget integrity testing!

I worked on a system whose Ruby was on version 2.6.7 and needed to be updated to at least the next version because it was hosted on Heroku and the container that worked with this version would no longer be supported in 2 months (and in a few months more it would no longer be available for deployment). At the time I joined this project there was not one line of test written, and we didn't have a QA team yet. It took 2 complicated months to write a minimum of test coverage so that we would be able to carry out the version update the Ruby and Rails versions with more confidence.

# Conclusion

While gems/libraries undoubtedly offer valuable functionality and convenience in Ruby on Rails development, their indiscriminate usage can introduce numerous challenges and risks. By prioritizing self-contained code within the repository, developers can minimize version incompatibilities, simplify maintenance efforts, mitigate security risks, optimize performance, and avoid dependency bloat. Ultimately, exercising restraint in the adoption of external dependencies promotes greater code stability, security, and maintainability, ensuring the long-term success of Rails applications.
