---
layout: post
title:  "Think About Your Audience!"
date:   2022-07-04 16:30:00 +0530
categories: jekyll update
---

Have you ever wondered if you left your work station without shutting down
laptop and next time when you start working, everything is as you left....?

Hi everyone, I am Sonali Srivastava, an Outreachy Intern and contributor to
FOSS project systemd.

So i have question for you, do you get confused when you see options like
suspend, sleep, hibernate, suspend-then-hibernate, hybrid sleep and more
depending on which OS you have, the names may vary....?

## I do!

Or you can say I did before i started working on systemd project!

Let's get started...

**Power Management with Systemd**

- **suspend[aka sleep]** : With the **lowest resume time**, this requires
  continues power supply or utilises battery **[low power consumption]**
  but system's state loss is possible if system runs out of battery
  backup or power. This mode saves system's state in **RAM**.

- **hibernate** : Compared to suspend, this has **higher resume time**, but this
  requires **no power consumption** and system's state loss is not possible as
  it is stored in the **HARD DISK DRIVE**.

- **hybrid-sleep[aka suspend-to-both]** : Like the name suggets, its the
  combination of above two. The system's state is saved in **RAM** as well as
  **HARD DISK DRIVE**. Since system is suspended so there is **lower resume
  time** and also the system's state is not loss in case of battery or power
  loss.

- **suspend-then-hibernate** : This is an interesting one. Till now, the
  feature was to suspend initially for duration mentioned in
  **HibernateDelaySec** [which defaults to 2 hours] and then hibernate
  but after the project i have been assigned by systemd, i have worked
  on improving this feature. But let's first know how it initially worked.

  Compared to above three, this has different benefit and risk. System is
  suspended for some duration being set by user ir default so if battery does not
  die in that duration then state is saved in **HDD** and system hibernates. The
  **resume time is low** but if system dies during suspend due to power outage
  battery exhaust, system's state is lost.

## So what did i do[/still doing/will do] ?

- Enabled check for battery's charge level before initiating suspend.

- Enabled battery consumption rate estimation after wakeup from suspend.

- Enabled timed wakeup to do the assess again.

- Enabled support for multiple batteries.

- Record battery consumption rate on machine specific keys.

- Handle when to wakeup for the first check.

- Enabled hardware alarm trigger to be used while suspending.

.... and more.

Looking forward for you test the feature update i have given. In case you are
interested in trying, next time when you are temporarily away from system,
instead of suspend, try the following :

```
$systemctl suspend-then-hibernate
```

You can also check ```journalctl``` to see the 'systemd-sleep' logs.

Happy reading.

Feel free to drop your queries on : srivastava.sonali1@gmail.com
