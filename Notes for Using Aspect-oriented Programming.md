### Notes for Using Aspect-oriented Programming


##### 1. A traditional way to measure the execution time of one function
```
long start = System.currentTimeMillis();
myfunction()
long end = System.currentTimeMillis();
logger.info(“myfunction takes “ + (end-start) + “ ms.”);	
```

  Analogy: you read your electricity meter at the beginning and the end of each month, report the usage to the electricity company.

  We should focus on our core functionality, and leave electricity meter reading task (an aspect)  to someone else. 

##### 2.  AOP allows us to add execution blocks without explicitly changing the source code of the target class  
Let’s use an example to illustrate AOP program[1].  Suppose we have a class Performance:

```
package concert;
Public interface Performance{
	Public void perform();
}
```

Another class Audience interacts with the Performance: take seats, silence cell phones before perform(); give an applause or demand a refund after perform().  Certainly, we can insert code into Performance.perform() in the traditional way.  However, the Audience is not a part of the Performance.  The Audience is an aspect that can be applied to Performance.

```
package concert;
public aspect Audience{
	// define a named pointcut -- performance()
  	pointcut watchPerformance(): (“execution(* concert.Performance.perform(..))”)

  	// define an advice method
  	Object around(): watchPerformance(){
  		try{
  			System.out.println(“Silencing cell phones”);
			System.out.println(“Taking seats”);
	
        		//calling Performance.perform()
			Object result = proceed(); 
	
			System.out.println(“Clap Clap”);
		} catch(Throwable e){
			System.out.println(“Demanding a refund”);
		}
		Return result
	}
}
```

We can also use annotations to write the above WatchPerformance aspect.  In some very special situations, there are limitations for AspectJ code written in annotation style.[11]

```
package concert;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public aspect Audience{
	// define a named pointcut -- performance()
	@Pointcut(“execution(* Concert.Performance.perform(..))”)
	
	public void performance(){}

	// define an advice method
	@Around("performance()")

   	public void watchPerformance(ProceedingJoinPoint jp){
		try{
			System.out.println(“Silencing cell phones”);
			System.out.println(“Taking seats”);
			
			//calling Performance.perform()
			jp.proceed();
			
			System.out.println("Clap  Clap");
		}catch(Throwable e){
			System.out.println(“Demanding a refund”);
	   }
   }
```

##### 3. New Concepts
In the above example, we see four items:

A join point -- concert.Performance.perform()

An aspect -- class Audience

A pointcut -- watchPerformance(): (“execution(* concert.Performance.perform(..))”)

An advice -- watchPerformance();

There are several types of pointcuts: execution(), call(), target(), within(), etc.[14]

There are 5 kinds of advice: after(), after() returning, after() throwing, around(), and before()[2], or @After, @AfterReturning, @AfterThrowing, @Around, and @Before.

##### 4.Handling parameters in advice
The example in section 2 is completely static.  What if we want the advice take different actions according to the parameter of the join point?  Suppose we have a class

```
public class Disc{
	private String title;
	Private String artist;
	Private List<String> tracks;

	…… …… 

	Public void play(String trk){
		System.out.println(“Playing “ + title + “ by “ + artist + “-Track: “ + trk;
	}
}
```

If we want to know how many times each track are played, traditionally we have to modify the Dise class, or add code whenever Disc.play() is called.  With AOP, we can define a new aspect TrackCount:

…… ……

```
public aspect TrackCounter{
	private Map<String, Integer> trackCounts = new HashMap<String, Integer>();
	pointcut trackPlayed(): (“execution(* Disc.play(String)) ”);

	before(): trackPlayed(){
		Object[] args = thisJoinPoint.getArgs();
     		String track = (String)args[0]
		int currentCount = getPlayCount(track);
		trackCounts.put(track, currentCount+1);
	}

	public int getPlayCount(String track){
		return trackCounts.containsKey(track) ? trackCOunts.get(track) : 0;
	}
}
```

 We can also use annotations to write the above TrackCounter aspect.

```
…… ……
import org.aspectj.lang.annotation.Before;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;

@Aspect
public class TrackCounter{
	Private Map<String, Integer> trackCounts = new HashMap<String, Integer>();
	
	@Pointcut(“exection(* Disc.play(String)) && args(track)”)
	
	Public void trackPlayed(String track);

	@Before(“trackPlayed(track)”)
	public void countTrack(String track){
		int currentCount = getPlayCount(track);
		trackCounts.put(track, currentCount+1); 
	}

	public int getPlayCount(String track){
		Return trackCounts.containsKey(track) ? trackCOunts.get(track) : 0;
	}
}
```

##### 5. Why can an Aspect monitor a target class?

AspectJ weavs a target class with aspects into a new class, which contains code from the aspects and the original target class.  The weaving can happen at compile time, class load time, or run-time.

Unlike Spring AOP, AspectJ only supports compile time weaving (CTW) and class load time weaving(LTW)[7]. CTW takes the target class (source code, or jar file) and the aspect, and produces a woven class file at compile time.  LTW weaves existing class files or JAR files with the aspect when a class loader loads the class files to the JVM.[7]. 

To utilize AspectJ, we need two AspectJ jar files: aspectjrt.jar (AspectJ run-time), and aspectjweaver (AspectJ Weaver, only used for load-time weaving (LTW))

To enable AspectJ LTW, one common way is to specify a javaagent option to the JVM: -javaagent:pathto/aspectjweaver.jar.

You can find an example for Spring AOP using AspectJ LTW[8]


##### 6. Other useful information

AspectJ is more powerful than Spring Framework AOP since AspectJ allows Pointcuts on constructors[3], and Pointcuts on field access[4], and AspectJ is known to be 8 to 35 times faster than Spring AOP[7].

AOP load-time weaving does not work for JDK classes (e.g. java.sql.Driver) because those classes are always loaded before the weaving agent.  The only workaround is to weave JDK distribution JAR (e.g rt.jar) into another JAR.[11],[12],[13], which usually is not recommended.

AOP has performance impact.  In my work, I observed the woven JDBC driver is 10% slower on average than the origial JDBC driver.


##### 7. References
[1] https://www.baeldung.com/aspectj

[2]https://o7planning.org/en/10257/java-aspect-oriented-programming-tutorial-with-aspectj

[3]https://stackoverflow.com/questions/17338788/aspectj-pointcut-on-constructor-object

[4]https://stackoverflow.com/questions/29561306/aspectj-how-to-get-accessed-fields-value-in-a-get-pointcut

[5]https://stackoverflow.com/questions/5656554/getting-a-return-value-or-exception-from-aspectj

[7]https://www.baeldung.com/spring-aop-vs-aspectj

[8]https://stackoverflow.com/questions/17008819/spring-aop-using-aspectj-ltw-not-working

[10] https://stackoverflow.com/questions/25299687/use-aspectj-to-monitor-database-access-methods

[11]http://www.zhuwu.me/blog/posts/use-aspectj-to-modify-java-standard-library

[12]https://stackoverflow.com/questions/42901291/does-aspectj-support-modifying-the-jdk-bytecode

[13]http://aspectj.2085585.n4.nabble.com/LTW-and-rt-jar-td2081407.html

[14]https://www.eclipse.org/aspectj/doc/next/progguide/language-joinPoints.html
