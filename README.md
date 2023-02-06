## @AfterReturning Advice Type
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
**創建**
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
