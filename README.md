## @AfterReturning Advice Type
>@AfterReturning是一種通知類型，可確保方法執行成功後運行通知

**創建Account.java實體類別**
```
public class Account {
	
	private String name;
	private String level;
	
	public Account() {
		
	}
	
	public Account(String name, String level) {
		this.name = name;
		this.level = level;
	}

	public String getName() {
		return name;
	}
	
	public void setName(String name) {
		this.name = name;
	}
	
	public String getLevel() {
		return level;
	}
	
	public void setLevel(String level) {
		this.level = level;
	}

	@Override
	public String toString() {
		return "Account [name=" + name + ", level=" + level + "]";
	}
}
```
**創建AccountDAO.java DAO類別，添加findAccounts方法，簡易添加成員並回傳最終數據**
```
@Component
public class AccountDAO {

	private String name;
	private String serviceCode;
	
	// add a new method: findAccounts()
	
	public List<Account> findAccounts(){
		
		List<Account> myAccounts = new ArrayList<>();
		
		// create sample accounts
		Account temp1 = new Account("John", "Silver");
		Account temp2 = new Account("Madhu", "Platinum");
		Account temp3 = new Account("Luca", "Gold");
		
		// add them to our accounts list
		myAccounts.add(temp1);
		myAccounts.add(temp2);
		myAccounts.add(temp3);	
		
		return myAccounts;
	}
}
```
**創建主應用程序AfterReturningDemoApp，執行findAccounts方法並添加一些訊息打印，以便查看整體AOP與主應用程序之間的運作關係**
```
public class AfterReturningDemoApp {

	public static void main(String[] args) {
		
		// read spring config java class
		AnnotationConfigApplicationContext context = 
				new AnnotationConfigApplicationContext(DemoConfig.class);
		
		// get the bean from spring container
		AccountDAO theAccountDAO = context.getBean("accountDAO", AccountDAO.class);
		
		// call method to find the accounts
		List<Account> theAccounts = theAccountDAO.findAccounts();
		
		// diaplay the accounts
		System.out.println("\n\nMain Program: AfterReturningDemoApp");
		System.out.println("----");
		
		System.out.println(theAccounts);
		
		System.out.println("\n");		
		
		// close the context
		context.close();
	}
}
```
**創建切入點聲明**
>參考網址：https://www.cnblogs.com/liaojie970/p/7883687.html
>
>Pointcut的定義包括兩個部分：
>* Pointcut表示式(expression) 
>```
>Ex:
>@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
>```
>* Pointcut簽名(signature) 
>```
>Ex:
>@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
>public void forDaoPackage() {} // Name of pointcut declaration
>```
>```
>@Pointcut("forDaoPackage()")
>public void forDaoPackageNoGetterSetter() {}
>```
>Pointcut定義時，還可以使用&&、||、! 這三個運算
>```
>@Pointcut("forDaoPackage() && !(getter() || setter())")
>public void forDaoPackageNoGetterSetter() {}
>```
>
**LuvAopExpressions.java完整程式碼，創建切入點聲明**
```
@Aspect
public class LuvAopExpressions {

	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.*(..))")
	public void forDaoPackage() {} // Name of pointcut declaration
	
	// create pointcut for getter methods
	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.get*(..))")
	public void getter() {}
	
	// create pointcut for setter methods
	@Pointcut("execution(* com.luv2code.aopdemo.dao.*.set*(..))")
	public void setter() {}
	
	// create pointcut: include package ... exclude getter/setter
	@Pointcut("forDaoPackage() && !(getter() || setter())")
	public void forDaoPackageNoGetterSetter() {}
}
```
>參考網址：https://juejin.cn/post/6844903969433583624
>
>**不同aspect，advice的執行順序**
>
>Spring AOP通過指定aspect的優先級，來控制不同aspect，advice的执行顺序，有兩種方式：
>
>* Aspect 類添加註解：org.springframework.core.annotation.Order，使用註解value屬性指定優先級。
>
>* Aspect 類實現接口：org.springframework.core.Ordered，實現Ordered 接口的getOrder()方法。其中，數值越低，表明優先級越高，@Order默認為最低優先級，即最大數值：
```
  /**
	 * Useful constant for the lowest precedence value.
	 * @see java.lang.Integer#MAX_VALUE
	 */
	int LOWEST_PRECEDENCE = Integer.MAX_VALUE;
```
>最終，不同aspect，advice的执行顺序：
>
>* 入操作（Around（接入點執行前）、Before），優先級越高，越先執行；
>* 一個切面的入操作執行完，才輪到下一切面，所有切面入操作執行完，才開始執行接入點；
>* 出操作（Around（接入點執行後）、After、AfterReturning、AfterThrowing），優先級越低，越先執行。
>* 一個切面的出操作執行完，才輪到下一切面，直到返回到調用點
>![image](https://user-images.githubusercontent.com/101872264/217259290-3e42adda-07de-40f2-97e2-5946f7403b4b.png)

**創建MyApiAnalyticsAspect.java，此Aspect的優先順序為最後`@Order(3)`；同時套用LuvAopExpressions中所建立的切入點聲明**
```
@Aspect
@Component
@Order(3)
public class MyApiAnalyticsAspect {
	
	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void performApiAnalytics() {
		
		System.out.println("\n=====>>> Performing API analytics");
	}
}
```
>參考網址：https://www.cnblogs.com/kenx/p/15093310.html
>
>**JoinPoint對象**
>
>JoinPoint對象封裝了SpringAop中切面方法的信息,在切面方法中添加JoinPoint參數,就可以獲取到封裝了該方法信息的JoinPoint對象.
>![image](https://user-images.githubusercontent.com/101872264/217263978-e2a73f62-4790-4258-81a5-3fc5c46e4911.png)

**此Aspect為演示@AfterReturning功能，優先順序為次要`@Order(2)`，在指定方法執行前透過beforeAddAccountAdvice(`@Before`)顯示完整方法名稱並列出相關攜帶參數以及屬性值；而afterReturningAccountsAdvice(`@AfterReturning`)則是在指定方法成功執行並返回數據時進入，透過而外方法convertAccountNamesToUpperCase將數據Name皆改為大寫字母，成為最終回傳數據**
```
@Aspect
@Component
@Order(2)
public class MyDemoLoggingAspect {

	// add a new advice for @AfterReturning on the findAccounts method
	@AfterReturning(
			pointcut="execution(* com.luv2code.aopdemo.dao.AccountDAO.findAccounts(..))",
			returning="result")
	public void afterReturningAccountsAdvice(
					JoinPoint theJoinPoint, List<Account> result) {
		
		// print out which method we are advising on
		String method = theJoinPoint.getSignature().toShortString();
		System.out.println("\n=====>>> Excuting @AfterReturning on method: " + method);
		
		// print out the results of the method call
		System.out.println("\n=====>>> result is: " + result);
		
		// let's post-process the data ... let's modify it
		// convert the account names to uppercase
		convertAccountNamesToUpperCase(result);
		
		System.out.println("\n=====>>> result is: " + result);
	}
	
	private void convertAccountNamesToUpperCase(List<Account> result) {
		// loop through accounts
		for (Account tempAccount : result) {
		
			// get uppercase version of name
			String theUpperName = tempAccount.getName().toUpperCase();
		
			// update the name on the account
			tempAccount.setName(theUpperName);
		}
	}

	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void beforeAddAccountAdvice(JoinPoint theJoinPoint) {
		
		System.out.println("\n=====>>> Excuting @Before advice on method()");
		
		// display the method signature
		MethodSignature methodSig = (MethodSignature) theJoinPoint.getSignature();
		
		System.out.println("Method: " + methodSig);
		
		// display method arguments
		// get args
		Object[] args = theJoinPoint.getArgs();
		
		// loop thru args
		for (Object tempArg : args) {
			System.out.println(tempArg);
			
			if (tempArg instanceof Account) {
				
				// downcast and print Account specific stuff
				Account theAccount = (Account) tempArg;
				
				System.out.println("account name: " + theAccount.getName());
				System.out.println("account level: " + theAccount.getLevel());
			}
		}
	}
}
```
**此Aspect為最優先順序`@Order(1)`，透過不同訊息方便查看不同Aspect之間的作用順序；同樣引用了LuvAopExpressions中的切入點聲明**
```
@Aspect
@Component
@Order(1)
public class myCloudLogAsyncAspect {
	
	@Before("com.luv2code.aopdemo.aspect.LuvAopExpressions.forDaoPackageNoGetterSetter()")
	public void logToCloudAsync() {
		
		System.out.println("\n=====>>> Logging to Cloud in async fashion");
	}
}
```
![image](https://user-images.githubusercontent.com/101872264/217265349-8a3db7cf-dcb1-4e35-b6f1-3137e353d3f3.png)
