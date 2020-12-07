+++
"author" = "Rizin"
"title" = "Frequently Asked Questions"
"date" = "2020-12-05"
"summary" = "Who are you? Why did you fork radare2? What will happen to Cutter now? Our answers to your frequently asked questions."
"tags" = ["rizin"]

"ShowToc" = true
"TocOpen" = false
"weight" = 2
+++

## Who are you?

We are a group of developers and security enthusiasts who contributed to radare2 in one way or the other in the past years. Some of us got involved with radare2 up to 8 years ago. We were, together with pancake — the original author — the maintainers of the radare2 project. We developed, handled issues, pull requests, review, CI and more. Some of us are the team who lead and maintain the Cutter project, a popular Graphic User Interface for the radare2 project. Among others, we started the development and integration of popular decompilation plugins for radare2 such as r2ghidra and r2dec.

---

## Why did you fork radare2?

During the years, the direction that radare2 was led to was not aligned with what we believed is the best for the project and the community. These disagreements covered many of the aspects involved in creating an open source project — technical, interpersonal, and managerial.

Our efforts and tries to change and improve things partially fell on deaf ears. With time, the environment that was created was one where many of us felt stressed, disrespected, and unwelcome. An environment that for years affected users, contributors and core members that eventually decided to leave because of it.

Radare2 as a project evolved and couldn't anymore be treated as a toy tool. With the number of users growing every year we are in the ultimate responsibility to provide them a stable, usable framework. As the core developer team, we have come to the conclusion that it is impossible for us to continue to pursue the goal of making radare2 better under the current circumstances and environment.

It is natural for Open Source projects to separate to different journeys with different visions. We all want to participate and contribute to projects we are passionate about, which we believe in, feel safe and welcome, and enjoy working on. For the aforementioned reasons and others, we believe that it is better for us to move forward on our own and fork the project.

---

## What are the differences between Rizin and radare2?

Rizin is a new born project that was created from radare2, hence more and more changes and differences will appear over time. With the establishment of Rizin, we are committed to create an environment and a project which will be aligned with our values and vision for an open source project and community.

We see it as our ultimate responsibility to provide the users with a stable and usable program that they can rely on. We will put efforts on releasing stable versions of Rizin and improving our test suite.

It is also in our obligation to create an environment where developers, contributors and users feel welcome and safe. For this, we put in place multiple instruments that will allow us to enforce such behavior. We adopted the Contributor Covenant Code of Conduct as we believe it is aligned with our values and with the community we want to create around Rizin. We will follow the code of conduct and enforce it on our different platforms. We started efforts of cleaning the source code from phrases that are can't be part in the environment we want to create. In addition, we will put efforts in creating a more inclusive and diverse community and welcome new contributors.

Technically speaking, Rizin already contains many changes that do not exist in radare2. Some of them are noted below:

- **New Projects:** we replaced the existing project functionality with a new one, developed entirely from scratch, that is based on serialization of existing objects instead of replication of commands. A blog post about this new feature will soon be published, so stay tuned if you want to know more!
- **Removal of less tested/stable features:** As we strive to provide a stable tool that you can trust, we chose to remove some features that we believe are not widely used, are old or are not tested at all and thus do not provide any value in their current state. This includes features such as the embedded WebUI, `m` commands, old projects, the `pdc` command, `T` commands, and others.
- **Switch to Git submodules instead of copy-pasted code:** this will allow us to better track the external code used in Rizin.
- **Deprecation of ACR/Makefile build system in favor of Meson:** experience has shown that a more declarative approach as used by Meson is easier to maintain and understand. Although at the moment the ACR/Makefile build system contains some features that Meson in Rizin is missing, it is also slow (in terms of compilation time), complicated to edit and does not support out-of-source builds. If more additions will be needed, we will be able to implement them in Meson.
- **New shell behavior and overall commands handling:** We recently developed in radare2 a new way to parse user commands, register them and develop them. This feature is called `cfg.newshell` and it will both make the user experience more consistent and the developer experience smoother. For these reason we have improved and enabled this by default in Rizin. We will publish a separate blog post about this soon!

---

## What will happen to radare2 now?

We don't know. radare2 is a popular project with many contributors and users. The maintainer of radare2 will decide how things will proceed. Such a big move will naturally cause changes and we wish to work together to resolve them while causing the least amount of discomfort to the members of the radare2 community and the users.

We wish the radare2 project the best of luck. 

---

## What about Cutter?

The Core team of Cutter, who were also a part of radare2 Core team, left radare2 and co-founded Rizin. Following this, Cutter is switching from radare2 to Rizin as its backend. For the users of Cutter, nothing major should change. Development on Cutter will continue as usual. Changes in the organization and policies (e.g Code of Conduct) will also apply to Cutter. Radare2 may or may not fork Cutter back to support radare2 instead and that is up to the radare2 maintainers.

---

## Will you contribute to radare2?

As we are forking radare2, we would stop the contribution to the original project, though we expect patches to be imported from one project to the other for some time. In some cases, like a discovery of security vulnerabilities in mutual code, we would love to notify the radare2 team so users of the project will be protected.

---


## Can I take part and contribute to Rizin?

Absolutely! We are thrilled to help you start and join Rizin. Please read our initial [documentation for new contributors](https://github.com/rizinorg/rizin/blob/dev/CONTRIBUTING.md). Please join our [Mattermost chat](im.rizin.re/) or [#rizin-dev IRC channel](https://webchat.freenode.net/?channels=#rizin-dev) on freenode! We hope to create better on-boarding guides for new contributors in the coming months, but for the meantime, we are here for any question you have.

---

## What actions will you make to keep Rizin a safe environment for contributors and users?

The Rizin organization believe that contributors, developers and users should enjoy their time around the community and feel safe an welcome. We adopted a Code of Conduct that we believe is aligned with our values and with the community we want to create around Rizin. We will enforce it on our different platforms.

We started efforts of cleaning the source code from offensive phrases and comments. In addition, we will put efforts in creating a more inclusive and diverse community and welcome new contributors.

Finally, we created the concept of teams that will be responsible for different aspects of Rizin. Such teams will also include a Community team that, among other things, will be an address for requests and complaints from community members. 

---

## What is the future of Rizin?

We intend to make Rizin a stable project you can trust for your reverse engineering tasks and a welcoming environment where people can work together on something they care. We will release a roadmap with the features we want to work on and the direction we will take. In the short run, you can expect refinements to the new projects and to the shell.

---

## I have more questions, where can I ask?

We would love to answers your question. You can send us a message on [Mattermost](https://im.rizin.re) or [email](mailto:core@rizin.re) us. Please note that we do not gurantee to answer all questions, as some topics are personal or we prefer to keep for ourselves.

---

