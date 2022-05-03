## 1. 数据类型
    Integer number = 1000

    Boolean shipmentDispatched; 
    shipmentDispatched = true;

    Date shipmantDate = date.today();

    Long companyRevenue = 2394376475930374;

    Account objAccount = new Account (Name = 'Test');

    String companyName = 'test company';

    Time

    Blob

    Enum

    # declearing an sObject variable of type Account
    Account objName = new Account();
    # assignment of values to fields of sObject
    objAccount.Name = 'test customer'
    objAccount.Description = 'some descriptions'


## 2. 字符串

    # contains()
    String productName = 'thc';
    String productNameCopy = 'thcuande';
    procompr = productNameCopy.contains(productName)

    # equals()
    # equalsIgnoreCase()
    # remove()
    # removeEndIgnoreCase()
    # startWith()

## 3. 数组

    # define an array
    String [] = arrayOfProducts The new List <String> ();
    # add element
    arrayOfProducts.add('HCL')
    arrayOfProducts.add('H2SO4')
    arrayOfProducts.add('NACL')

## 4. 常量

定义在整个程序执行过程中应该具有常量值的变量，用final关键字声明。

    public class CustomerOperationClass {
        static final Double regularCustomerDiscount = 0.1;
        static Double finalPrice = 0;
        public static Double provideDiscount(Integer price){
            // calculate the discount
            finalPrice = price - price*regularCustomerDiscount;
            return finalPrice
        }
    }

    Double finalPrice = CustomerOperationClass().provide(100);

## 5. 决策

    if
    if ... else
    if ... elseif ... else

## 6. 循环

## 7. 集合
### List

    // string列表
    List<string> ListOfCities = new List<string>();
    //赋初始值
    List<string> ListOfCities = new List<string>{'NY','LA','LV'};
    //账户列表
    List<account> AccountToDelete = new List<account>();

    //List方法
    size()
    add()
    get()
    clear()
    set()

### Set

    Set<string> ProductSet = new Set<string>{'Phenol', 'Benzene', 'H2SO4'};
    // Set方法
    add()
    remove()
    contains()

### Maps

    Map<string, string> ProductCodeToProductName = new Map<string, string> {'1000'=>'HCL', '1001'=>'H2SO4'};
    //Map方法
    put()
    containsKey()
    get()
    keySet()

## 8. 类

    public class MySampleApexClass{
        public static Integer myValue = 0;
        public static String myString = '';
    
        public static Tnteger GetCalculateValue(){
            myValue = myValue + 10
            return myValue
        }
    
    }

### 访问修饰符
Private/Public/Global

### 共享模式
共享 "With Sharing"/无共享/虚拟"virtual"

### 类变量
public/private/protected/global/final/statistic
- 类变量数据类型和类变量名必须
- 访问修饰符与变量值可选

## 9. 类方法

    //Method definition and body
    public static Integer getCalculatedValue () {
        //do some calculation
        myValue = myValue+10;
        return myValue;
    }
- 类方法修饰符 public/protected
- 返回类型必须，无返回值则void

### 类构造函数
- 类构造函数是初始化对象时调用的函数，与类具有相同的名称
- 默认情况下调用无参构造方法
- 当在类初始化时想要完成变量和过程的初始化时，构造函数发挥作用


    //Class definition and body
    public class MySampleApexClass2 {
        public static Double myValue;   //Class Member variable
        public static String myString;  //Class Member variable
            
        public MySampleApexClass2 () {
            myValue = 100;  //initialized variable when class is called
                
        }
            
        public static Double getCalculatedValue () {    //Method definition and body
            //do some calculation
            myValue = myValue+10;
            return myValue;

### 构造方法重载

## 10. 对象

    //Sample Class Example
    public class MyClass {
        Integer myInteger = 10;
        public void myMethod (Integer multiplier) {
            Integer multiplicationResult;
            multiplicationResult=multiplier*myInteger;
            System.debug('Multiplication is '+multiplicationResult);
        }
    }
    
    //Object Creation
    //Creating an object of class
    MyClass objClass = new MyClass();
    
    //Calling Class method using Class instance
    objClass.myMethod(100);

### sObject创建

    //standard sObject
    Account objAccount = new Account();
    objAcount.Name = 'tester';
    objAcount.Description = 'tester description';
    insert objAccount;
    
    //custom sObject
    APEX_Customer_c objCustomer = new APEX_Customer_c ();
    objCustomer.Name = 'ABC Customer';
    objCustomer.APEX_Customer_Decscription_c = 'Test Description';
    insert objCustomer;

### 静态变量
当加载类时，静态方法和变量只初始化一次。

    //Sample Class Example with Static Method
    public class MyStaticClass {
        Static Integer myInteger = 10;
        public static void myMethod (Integer multiplier) {
            Integer multiplicationResult;
            multiplicationResult=multiplier*myInteger;
            System.debug('Multiplication is '+multiplicationResult);
        }
    }
    
    //Calling the Class Method using Class Name and not using the instance object
    MyStaticClass.myMethod(100);

## 11. 接口

    //Interface
    public interface DiscountProcessor{
    Double percentageDiscountTobeApplied();//method signature only
    }

### 批处理的标准salesforce接口

## 12. DML
执行对数据库的修改

    insert objCust
    update updatedInvoiceList
    upsert //执行更新操作，若记录不存在则创建新记录
    delete
    undelete

## 13. SOSL & SOQL
SOSL: 在多个对象上搜索特定字符串

    FIND 'ABC*' IN ALL FIELDS RETURNING APEX_Invoice__c (Id,APEX_Customer__r.Name), Account];

SOQL: 在单个对象搜索
    
    SELECT Id, Name, APEX_Customer__r.Name, APEX_Status__c FROM APEX_Invoice__c WHERE createdDate = today
    //给两个对象建立联系
    objInvoice.APEX_Customer__c = [SELECT id FROM APEX_Customer__c WHERE Name = 'Customer Creation Test' LIMIT 1].id;
### 获取子记录
    List<apex_customer__c> ListCustomers = [SELECT Name, Id, (SELECT id, Name FROM Invoices__r) FROM APEX_Customer__c WHERE Name = 'ABC Customer'];//Query for fetching the Child records along with Parent
### 获取父记录
    ListOfInvoicesWithCustomerName = [SELECT Name, id, APEX_Customer__r.Name FROM APEX_Invoice__c LIMIT 10];    //Fetching the Parent record's values
### 聚合函数
    AggregateResult[] groupedResults = [SELECT AVG(APEX_Amount_Paid__c)averageAmount FROM APEX_Invoice__c WHERE APEX_Customer__r.Name = 'ABC Customer'];
    MIN()
    MAX()
### 绑定Apex变量

## 13. 触发器
    trigger triggerName on ObjectName (trigger_events) { Trigger_code_block }
### 可触发触发器的事件
    insert
    update
    delete
    merge
    upsert
    undelete
### 上下文变量
    trigger.new
    trigger.old
    trigger.newMap
    trigger.oldMap

## 14. 测试

    @isTest //声明测试类
    testMethod关键字 //在测试类中声明测试方法 
    static testMethod void myUnitTest() {
    SSystem.assert()
