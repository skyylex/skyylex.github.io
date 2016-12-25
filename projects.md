---
layout: default
---

### **Projects**
<br>

---------------------------------------------------------------------------------------------------

<br>
### **Luminous**
<br>
[[Related post](2016/05/21/luminous_at_early_stages.html)]
[[Source code](https://github.com/skyylex/Luminous-proof-of-concept)]

**Description**: Proof of concept was written in a Python with the main idea to create a tool for visualizing algorithms in an automatic way. The user has an implementation of some algorithm in Python and input data to run it, this information is transmitted to the utility that build a visual interpretation of performing this specific algorithm with this specific input. Why it's needed? I believe that idea of the algorithm's usually easier to explain when you have something to show	 your audience.

**Current state**: proof of concept was started on a very broad inputs and unspecified requirements. It became obvious that it cannot cover all needs. There is a necessity to limit what tool allows to do. And without such specification, there is no sense to move further. The project was put on hold to analyze and define a scope to work on.

---------------------------------------------------------------------------------------------------

<br>
### **ReactiveBeaver**
<br>
[[Source code](https://github.com/skyylex/ReactiveBeaver)]

**Description**: ePub parser written Objective-C using reactive-functional paradigm using ReactiveCocoa 2. Some time ago I was attracted by opportunities of RFP approach, precisely speaking RFP's relationship to functional approach and pure functions. I decided to give a try with some task suitable for functional (reactive-functional) approach and parsing was a good choice, because of data flows here are the main part of the logic. And using RFP approach is natural here.

**Current state**: the foundation of the project is almost complete. The main issue is an organizing accurate work as a framework target to share it with people. There is a need in some structural changes in the code to solve this. Temporaly paused to find time to do it.

---------------------------------------------------------------------------------------------------

<br>
### **SKEPReader**
<br>
[[Source code](https://github.com/skyylex/SKEPReader)]

**Description**: this project is about return to life an old ePub reader for iOS named AePubReader. Original source could be described as a demo project. It seemed like the author had no intent to use it in a production, instead he wanted to show the idea of creating ePub reader in iOS. So there were some issues with a code quality and design. Also it was rather old and was written in MRC.

**Current state**: Project could be launched and epub could be read - that's it. There are a lot of space for improvements and bug fixing, but I think there is no strong need to hurry up with the release of it. There are lot of good epub readers for use such as iBooks and Marvel. That's why I decided to use it as a place for additional programming and refactoring when I have time for this.