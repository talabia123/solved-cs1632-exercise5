Download Link: https://assignmentchef.com/product/solved-cs1632-exercise5
<br>
– [Exercise 5 – Static Analysis Part 1: Linters and Bug Finders](#exercise-5—static-analysis-part-1-linters-and-bug-finders)* [DrunkCarnivalShooter](#drunkcarnivalshooter)* [Applying SpotBugs and CheckStyle](#applying-spotbugs-and-checkstyle)+ [Lessons on Pattern Matching](#lessons-on-pattern-matching)* [Submission](#submission)* [Resources](#resources)– [Exercise 5 – Static Analysis Part 2: Model Checking](#exercise-5—static-analysis-part-2-model-checking)* [Applying Java Pathfinder (JPF)](#applying-java-pathfinder–jpf-)+ [Applying JPF on Rand](#applying-jpf-on-rand)+ [Applying JPF on DrunkCarnivalShooter](#applying-jpf-on-drunkcarnivalshooter)+ [Applying JPF on JUnit to Unit Test DrunkCarnivalShooter](#applying-jpf-on-junit-to-unit-test-drunkcarnivalshooter)+ [Lessons on Model Checking](#lessons-on-model-checking)* [Submission](#submission-1)* [GradeScope Feedback](#gradescope-feedback)* [Resources](#resources-1)

In this exercise, you will use two static analysis tools to test a program: abug finder named SpotBugs and a linter named CheckStyle, SpotBugs andCheckStyle work in similar ways in that both look for patterns that are eithersymptomatic of a bug (former) or are bad coding style (latter). So we willlook at them together first. Later in Part 2, we will do model checking usingthe Java Path Finder (JPF) which is much more rigorous in proving a programcorrect.

## DrunkCarnivalShooter

DrunkCarnivalShooter is a simple text-based game where the player goes to acarnival shooting range and tries to win the prize by shooting all 4 proviedtargets. The player can designate what target to shoot for pressing 0-3. Butsince the player is drunk, there is an equal chance of the player shooting leftor right as shooting straight. Refer to the file[sample_run.txt](sample_run.txt) for an example game play session. You canalso try playing it yourself using the reference implementation:

“`$ java -jar DrunkCarnivalShooter.jar“`

To run the DrunkCarnivalShooter using the current implementation (for Windows users):

“`$ run.bat“`

For Mac or Linux:

“`$ bash run.sh“`

Now the current implementation contains several bugs. In fact, the game throwsan exception immediately at start up:

“`$ java -cp bin;jpf-core/build/* DrunkCarnivalShooterImplException in thread “main” java.lang.NullPointerExceptionat DrunkCarnivalShooterImpl.&lt;init&gt;(DrunkCarnivalShooterImpl.java:31)at DrunkCarnivalShooterImpl.main(DrunkCarnivalShooterImpl.java:150)“`

In this exercise, we are going to try to debug the program using staticanalysis instead of dynamic testing. So now let’s go and try to find somedefects!

## Applying SpotBugs and CheckStyle

Try running both tools on DrunkCarnivalShooterImpl.java. As usual, I’veprovided scripts to run each tool.

To run CheckStyle (Windows users):

“`$ runCheckstyle.bat“`

To run CheckStyle (Mac/Linux users):

“`$ bash runCheckstyle.sh“`

To run SpotBugs (Windows users):

“`$ runSpotbugs.bat“`

To run SpotBugs (Mac/Linux users):

“`$ bash runSpotbugs.sh“`

CheckStyle and Spotbugs should print out several warnings each. Refer to thebelow references to find out what each warning means at fix your code at thecorresponding source code line:

* CheckStyle reference: https://checkstyle.sourceforge.io/checks.htmlIf you don’t understand a CheckStyle warning, read the corresponding entry inside google_checks_modified.xml under the checkstyle-jars folder and the above reference.

* SpotBugs reference: https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html* There is a GUI for SpotBugs if that is what you prefer. You can launch the GUI by using the following command:“`$ java -jar spotbugs-4.0.0-beta4/lib/spotbugs.jar“`The following link contains a short tutorial on how to use the GUI:https://spotbugs.readthedocs.io/en/latest/gui.html

SpotBugs will complain about among other things a warning type called“ST_WRITE_TO_STATIC_FROM_INSTANCE_METHOD”. Look up the meaning of this errortype in the above SpotBugs reference. It is saying that you should not updatea static variable from an instance method. I have seen many of you do this alot in your assignments. You would declare variables that should really beinstance variables to be static. I don’t know where you picked up thatprogramming habit, but that goes against all OOP principles. If you are stillunsure about when to use static and when to use instance variables, here is agood tutorial:

https://docs.oracle.com/javase/tutorial/java/javaOO/classvars.html

After removing all warnings, you should see the following ouput for each.

Checkstyle output:

“`$ java -jar checkstyle-jars /checkstyle-7.0-all.jar -c checkstyle-jars/google_checks_modified.xml src/DrunkCarnivalShooterImpl.javaStarting audit…Audit done.“`

SpotBugs output:

“`$ java -jar spotbugs-4.0.0-beta4/lib/spotbugs.jar -textui -low -effort:max -longBugCodes -exclude spotbugs-4.0.0-beta4/my_exclude_filter.xml bin/*.classThe following classes needed for analysis were missing:org.junit.runner.JUnitCoreorg.junit.runner.Resultorg.junit.runner.notification.Failure“`

The missing classes are JUnit library classes. We are not interested indebugging the JUnit library, so we did not pass it to SpotBugs.

After fixing all the warning, now the program should at least start up properly, when run with run.bat:

“`$ java -cp bin;jpf-core/build/* DrunkCarnivalShooterImplRound #0: || || || ||Choose your target (0-3):“`

Yay! But we are note done yet. There are still bugs remaining. Try repeatedly shooting the first target by choosing 0.

“`$ java -cp bin;jpf-core/build/* DrunkCarnivalShooterImplRound #0: || || || ||Choose your target (0-3):0

…

Round #3: || || ||Choose your target (0-3):0Exception in thread “main” java.lang.ArrayIndexOutOfBoundsException: -1at java.util.ArrayList.elementData(ArrayList.java:422)at java.util.ArrayList.get(ArrayList.java:435)at DrunkCarnivalShooterImpl.isTargetStanding(DrunkCarnivalShooterImpl.java:125)at DrunkCarnivalShooterImpl.takeDownTarget(DrunkCarnivalShooterImpl.java:108)at DrunkCarnivalShooterImpl.shoot(DrunkCarnivalShooterImpl.java:89)at DrunkCarnivalShooterImpl.main(DrunkCarnivalShooterImpl.java:158)“`

The bug does not manifest in a deterministic way due to the randomness but youwill trigger it soon enough. Or you may encounter another bug where the gameends prematurely even when there are targets remaining. These bugs are bugs inthe logic of the program and SpotBugs is not very good at finding these typesof bugs. It only finds bugs that match a certain pattern.

### Lessons on Pattern Matching

Both linters (CheckStyle) and bug finders (SpotBugs) work by pattern matching.Pattern matching can be good at finding simple bugs that are recurrent acrossprojects and can even catch errors in your documentation. What they are notgood for is finding problems in your program logic (as seen above). For that,you need dynamic testing that actually executes the program to check programbehavior. Or, you can use model checking that is able to *prove* that certaincorrectness properties hold for all situations, even for random programs likethis one (see next section).

## Submission

Each pairwise group will do one submission to GradeScope, by *one member* ofthe group. The submitting member will press the “View or edit group” link atthe top-right corner of the assignment page after submission to add his/herpartner.

You will create a github repository just for exercise 5, as usual. Add yourpartner as a collaborator so both of you have access. Make sure you keep therepository *PRIVATE* so that nobody else can access your repository. When youare done, submit your github repository to GradeScope at the **Exercise 5 Part1 GitHub** link. Once you submit, GradeScope will run the autograder to gradeyou and give feedback. If you get deductions, fix your code based on thefeedback and resubmit. Repeat until you don’t get deductions.

If you don’t get any more warnings you’ve done your job. Otherwise, it is -1point for each CheckStyle or SpotBugs warning.

## Resources

* CheckStyle reference:https://checkstyle.sourceforge.io/checks.htmlIf you don’t understand a CheckStyle warning, read the corresponding entry inside google_checks_modified.xml under the checkstyle-jars folder and the above reference.

* SpotBugs reference:https://spotbugs.readthedocs.io/en/latest/bugDescriptions.html

# Exercise 5 – Static Analysis Part 2: Model Checking

* DUE: Nov 4, 2020 09:00 AM (Mon/Wed class)* DUE: Nov 5, 2020 09:00 AM (Tue/Thu class)

In Part 2, you will use a model checker named Java Pathfinder (JPF) to provevarious correctness properties in your program.

* IMPORTANT: You need Java 8 (1.8.0.231, preferably) to run the Java PathFinder model checker. Make sure you have the correct Java version by doing“java -version” and “javac -version” before going into the JPF section. If youdon’t have the correct version, here is a link to a folder with installationpackages for each OS:

https://drive.google.com/drive/folders/1E76H7y2nMsrdiBwJi0nwlzczAgTKKhv7

## Applying Java Pathfinder (JPF)

Java Pathfinder is a tool developed by NASA to model check Java programs. Itworks in exactly the same way we learned in class: it does an exhaustive andsystematic exploration of program state space to check for correctness.

### Applying JPF on Rand

Let’s first try out JPF on the example Rand program we saw on “Lecture 16:Static Analysis Part 2″ slides:

&lt;img src=”jpf.png” width=”50%” height=”50%”&gt;

First cd into the Rand directory before executing the scripts.

To run the Rand program (for Windows users):

“`$ run.bat“`

To run JPF with Rand:

“`$ runJPF.bat Rand.jpf“`

For Mac or Linux users, please run the corresponding .sh scripts.

When you run Rand with JPF, you can see from the screen output that it goesthrough all possible states, thereby finding the two states with division-by-0exception errors (I configured JPF to find all possible errors). So, now weknow that there are two defective states, how do we debug? You will see thatJPF has generated a trace file named [Rand.trace](Rand/Rand.trace) of all thechoices it had made to get to that state. You will see two traces since thereare two defective states. Pay attention to “cur” value of each Random.nextIntinvocation (that is the choice JPF has made for that invocation). The firsttrace shows values of 0, 2 for a, b and the second trace shows cur values of 1,1 for a, b. These are exactly the values that would cause a division-by-0exception at c = a / (b + a – 2). In this way, the trace file lets you easily tracethrough the code to get to the defective state.

### Applying JPF on DrunkCarnivalShooter

Now let’s cd out of the Rand directory to the root directory to once again workon DrunkCarnivalShooter. The following script applies JPF toDrunkCarnivalShooter.

For Windows users:

“`$ runJPF.bat DrunkCarnivalShooter.win.jpf“`

For Mac/Linux users:

“`$ bash runJPF.sh DrunkCarnivalShooter.macos.jpf“`

If you run the above, JPF will immediately display an error similar to the following:

“`…====================================================== search started: 10/25/20 9:54 PMRound #0: || || || ||Choose your target (0-3):

====================================================== resultsno errors detected…“`

No errors, yay! So are we done? No, far from it. From the search output, youcan see that the search ended immediately at the first point of user input.That is because JPF is not designed to receive user input from the terminal.Instead, JPF uses a set of APIs under the Verify class (gov.nasa.jpf.vm.Verify)to specify the set of user input(s) we want to test. Then, it **exhaustivelytests the program for each user input**.

In order to be able to use this feature, we first have to import the class atthe top of DrunkCarnivalShooterImpl.java:

“`import gov.nasa.jpf.vm.Verify;“`

Then replace calls to Scanner with calls to Verify only when the commandlineargument “test” is passed to the program. The “test” argument will put theprogram in test mode and not in play mode. You can see “test” is alreadyconfigured as the commandline argument in the target.args entry in[DrunkCarnivalShooter.win.jpf](DrunkCarnivalShooter/DrunkCarnivalShooter.win.jpf).This will allow us to still play game the game in normal mode.

In test mode, do not create Scanner and instead of scanning user input usingthe following statement:

“`int t = scanner.nextInt();“`

replace it with the following:

“`int t = Verify.getInt(0, 3);“`

The above will direct JPF to generate 4 states each where t is set to 0, 1, 2,or 3 respectively. Then it will systematically explore each state. If youwish, you can test a larger set of numbers beyond 0-3. It is just going togenerate more states and take longer (the flipside being you will be able tomodel check your program against a larger set of inputs).

Now let’s try running runJPF.bat one more time like the above. This will showa new error message due to an exception:

“`====================================================== search started: 10/25/20 10:01 PMRound #0: || || || ||Choose your target (0-3):

====================================================== error 1gov.nasa.jpf.vm.NoUncaughtExceptionsPropertyjava.lang.ArrayIndexOutOfBoundsException: -1at java.util.ArrayList.elementData(java/util/ArrayList.java:422)at java.util.ArrayList.get(java/util/ArrayList.java:435)at DrunkCarnivalShooterImpl.isTargetStanding(DrunkCarnivalShooterImpl.java:124)at DrunkCarnivalShooterImpl.takeDownTarget(DrunkCarnivalShooterImpl.java:108)at DrunkCarnivalShooterImpl.shoot(DrunkCarnivalShooterImpl.java:89)at DrunkCarnivalShooterImpl.main(DrunkCarnivalShooterImpl.java:163)…“`

Wait, this is the same exception that we randomly experienced previously! Usethe trace generated as part of the output to find the input value(s) and therandom value(s) that led to the exception. Interpret it in the same way youdid Rand.trace. The trace should look like:

“`====================================================== trace #1—————————————————— transition #0 thread: 0gov.nasa.jpf.vm.choice.ThreadChoiceFromSet {id:”ROOT” ,1/1,isCascaded:false}[50072 insn w/o sources]—————————————————— transition #1 thread: 0gov.nasa.jpf.vm.choice.BreakGenerator {id:”MAX_TRANSITION_LENGTH” ,1/1,isCascaded:false}[48603 insn w/o sources]—————————————————— transition #2 thread: 0gov.nasa.jpf.vm.choice.IntIntervalGenerator[id=”verifyGetInt(II)”,isCascaded:false,0..3,delta=+1,cur=0][22 insn w/o sources]—————————————————— transition #3 thread: 0gov.nasa.jpf.vm.choice.IntIntervalGenerator[id=”verifyGetInt(II)”,isCascaded:false,0..2,delta=+1,cur=0][64 insn w/o sources]…“`

Now let’s try to break down that trace. A transition happens in the course oftravering the program state space. Whenever there is a “choice” between onemore program states, a transition is recorded with the selected choice. Whenis JPF presented with a choice?

1. When it encounters Verify.getInt, it is presented with a range of userinputs each of which represents a separate path that JPF can take. Samething applies to all the other Verify APIs.

1. When it encounters Random.nextInt, it is presented with a range of randomvalues that the random number generator can return, each of each againrepresents a different program state.

1. When the program is multithreaded (parallel), JPF is also presented at eachinstruction with a choice of whether to context switch and execute anotherthread. This is what transition #0 is. This is how JPF can exhaustivelyexplore all thread interleavings. If you don’t know what that means, don’tworry about it. It is beyond the scope of this class. Feel free to ask if youare curious :).

In the above trace, it is important to understand transitions #2 and #3. Whatwould be transition #2 with choice interval 0..3? It would be theVerify.getInt(0, 3) that replaced the scan of user input. And according to thetrace it returned 0 (“cur=0”). What would be transition #3 with choiceinterval 0..2? It would be the rand.nextInt(3) used to add randomness to theshooting target, and in the trace it also returned 0 (“cur=0”). So it’s thecase where the user chose target 0 and the randomness of the shooting pulledthe bullet to the left. What’s on the left side of target 0? That should helpyou track down the problem. **Hint: What happens when t becomes -1 in isTargetStanding?**

Once you fix these bugs, try running runJPF.bat one more time. Now that youhave fixed the buggy state JPF runs for much longer. In fact, JPF is going tofall into an infinite loop and generate an infinite number of states (observedby the ever increasing Round number).

“`…Round #20:|| || ||Choose your target (0-3):You aimed at target #0 but the Force pulls your bullet to the left.You miss! “Do or do not. There is no try.”, Yoda chides.

Round #21:|| || ||Choose your target (0-3):You miss! “Do or do not. There is no try.”, Yoda chides.

… (to infinity)“`

There is no theoretical limit to the number of rounds a player can play, hencethe state explosion. How can I deal with this explosion and still verify myprogram?

We have to somehow narrow down the amount of state we test, or we will beforced to but JPF off after testing only a limited set of rounds. Let’s saythe state that we are really interested in relation to the specifications isthe state of the 4 targets. Now if you think about it, the 4 targets can onlybe in a handful of states: 2 * 2 * 2 * 2 = 16 states (standing or toppled foreach). And this is true no matter how many rounds you go through. The onlything that constantly changes every round is the round number — and that isthe culprit leading to the state explosion. The round number is not somethingwe are interested in verifying right now. So, let’s filter that state out!

Import the appropriate JPF library at the top of DrunkCarnivalShooter.java again:

“`import gov.nasa.jpf.annotation.FilterField;“`

And now, let’s annotate roundNum such that it is filtered out:

“`@FilterField private int roundNum;“`

Now if we run runJPF.bat again, JPF will only go up to Round #2 and stop and declare “no errors detected”.

“`…Round #2:

Choose your target (0-3):

====================================================== resultsno errors detected“`

But why Round #2? We would expect that 4 rounds would be needed to cover allthe 16 possible states. In fact, if you see the output, you can see it doesnot cover all the possible 16 states. And somehow the game is able toterminate after 2 rounds. So the game now does not throw any exceptions butstill malfunctions.

JPF can only check for systems properties that it knows about while traversingthe program state space. If you don’t tell it anything, the only thing JPFknows about a Java program is that it should not throw exceptions at any point.If you want to check that your program behaves in a certain way according tothe requirements, you need to tell JPF about these additional properties. Theway to encode properties is though assertions that assert the given invariantproperty at that point of execution. There are two options to insert theseassertions:

1. The assertions be embedded in your program code in the form of Java assertstatements. This is useful in the context of systems testing your code asthese assertions will be checked while your system is running. But this hasthe drawback that your testing code is mixed in with your implementation codewhich is not good for code readability and/or maintenance. Also, it is hard toapply unit testing in a systematic way.

1. The other option is to use JPF as part of the JUnit testing framework.JUnit will check for defects by checking postconditions using JUnitassertions, just like with regular unit testing. But each of the JUnit testsbecome comprehensive and exhaustive thanks to JPF, even in the face of randomprogram behavior. JPF makes sure that the invariant assertion holds for allthe random behaviors the program can display. It also does this for allpossible user inputs if the Verify API is used.

We will choose the latter option.

### Applying JPF on JUnit to Unit Test DrunkCarnivalShooter

Now we are not systems testing DrunkCarnivalShooter. We want to invoke JUniton DrunkCarnivalShooter. The script to do that is as follows:

“`runJPF.bat JUnit.win.jpf“`

For Mac or Linux:

“`bash runJPF.sh JUnit.macos.jpf“`

If you peek into JUnit.win.jpf (or JUnit.macos.jpf), you will notice that nowthe execution target is set to TestRunner instead of DrunkCarnivalShooter:

“`target = TestRunner“`

TestRunner invokes JUnit on the DrunkCarnivalShooterTest test class. As is,DrunkCarnivalShooterTest.java is incomplete and does not do much. Fill in thelocations with // TODO comments inside DrunkCarnivalShooterTest.java. In thesetUp method, use the Verify API such that you enumerate all the 16 possiblestates that the game can be in, as well as the target choice made by the user(0-3). In this way, each of your JUnit test cases will be tested on allpossible states the game can be in with all possible user inputs.

In the testShoot() method, implement the preconditions, execution steps, andthe invariant to test the shoot(targetChoice, builder) method as explained inthe method comment. The invariant is a property that must hold no matter thegame state and the target choice. For this test, the invariant chosen was theremaining number of targets, because it appears that the game is endingprematurely thinking that there aren’t any more targets.

I recommend that you always insert the failString that I initialized for you inthe setUp method as the first argument of any JUnit assert call so that you getthat as part of your failure message. For example,

“`assertEquals(failString, expected value, observed value);“`

The failString tells you the combination of game state and target choice thatled to the failure, which helps you debug the problem. Feel free to appendadditional information to the failString that may help you debug.

If you implemented the test properly, you should see a long list of errors fordifferent combinations. Debug DrunkCarnivalShooterImpl to remove the errors.Now if you play the game, you should not see any defects.

### Lessons on Model Checking

What have we learned? We learned that a model checker such as JPF canguarantee correctness for the given set of inputs. But in order to do that,you often need to limit the amount of state JPF monitors to prevent stateexplosion. Also, the guarantee of correctness depends heavily on how much ofthe program specification you have encoded into your testing in the form ofassertions. If there are no assertions, JPF can only check only basic thingssuch as no exceptions.

## Submission

When you are done, submit your Exercise 5 github repository to GradeScope atthe **Exercise 5 Part 2 GitHub** link. Once you submit, GradeScope will runthe autograder to grade you and give feedback. If you get deductions, fix yourcode based on the feedback and resubmit. Repeat until you don’t getdeductions.

## GradeScope Feedback

The GradeScope autograder works in 2 phases:1. DrunkCarnivalShooterTest on DrunkCarnivalShooterImpl

This runs your DrunkCarnivalShooterTest JUnit test on yourDrunkCarnivalShooterImpl, with the help of JPF to do exhaustive stateexploration. Assuming your implementation is bug-free, it should not turn upany JUnit test failures.

1. DrunkCarnivalShooterTest on DrunkCarnivalShooterBuggy

This runs your DrunkCarnivalShooterTest JUnit test on the buggyDrunkCarnivalShooterBuggy implementation, again with the help of JPF. Sincethis implementation is buggy, JPF should find states where there are JUnit testfailures.

If you have trouble, try comparing your JPF outputs against these expectedoutputs:

* Result of running DrunkCarnivalShooterImpl standalone on top of JPF:“`runJPF.bat DrunkCarnivalShooter.win.jpf“`OR“`bash runJPF.sh DrunkCarnivalShooter.macos.jpf“`Expected output: [jpf_drunkcarnivalshooter_run.txt](jpf_drunkcarnivalshooter_run.txt)

* Result of running DrunkCarnivalShooterTest on DrunkCarnivalShooterImpl on top of JPF. Corresponds to the autograder phase 1 output:“`runJPF.bat JUnit.win.jpf“`OR“`bash runJPF.sh JUnit.macos.jpf“`Expected output: [jpf_junit_run.txt](jpf_junit_run.txt).

* Result of running DrunkCarnivalShooterTest on DrunkCarnivalShooterBuggy on top of JPF. Corresponds to the autograder phase 2 output. First uncomment “target.args = buggy” in JUnit.win.jpf or JUnit.macos.jpf, depending on your OS. Then run:“`runJPF.bat JUnit.win.jpf“`OR“`bash runJPF.sh JUnit.macos.jpf“`Expected output: [jpf_junit_buggy_run.txt](jpf_junit_buggy_run.txt).

Minor details like elapsed time statistics can differ but the search output andthe results output should look the same. Also, note that now the former goesup to Round #4 and covers all possible 16 target configurations.

## Resources

* JDK 8 installation packages:https://drive.google.com/drive/folders/1E76H7y2nMsrdiBwJi0nwlzczAgTKKhv7

* Java Path Finder manual:https://github.com/javapathfinder/jpf-core/wiki/How-to-use-JPFhttp://javapathfinder.sourceforge.net/

* Java Path Finder Verify API:https://github.com/javapathfinder/jpf-core/wiki/Verify-API-of-JPF