### @Binds(모듈 내 추상 메서드 + 하나의 매개변수)
* only it is used to provide interfaces, abstract classes or in your case classes that are extended. 
* 모듈 내 **추상 메서드 + 하나의 매개변수**에만 붙일 수 있다.
* 이미 바인드된 SecureRandom을 Random 타입으로 한 번 더 바인드할 수 있다.
  * Random이란 객체를 SecureRandom이란 객체에 바인딩
* @Provides 메서드보단 좀 더 효율적으로 사용할수 있다.
  * 코드상으로는 별 다른 차이가 없어보이지만 dagger 내부적으로 code를 생성할때는 @Binds를 사용하는 편이 훨씬 적은 코드를 생산해내고 효율성도 좋다 
  * 그렇지만 하나의 매개변수를 필요할기 때문에 무조건적으로 @Binds를 사용할수는 없다. 
  * 약 40%이상 코드가 절약된다.
* ```java
  @Binds
  abstract Random bindRandom(SecureRandom secureRandom)
* **사용이유는 코드를 간단하게 해준다.**
  * ```java
    @Provides
    public LoginContract.Presenter 
        provideLoginPresenter(LoginPresenter loginPresenter) {
            return loginPresenter;
    }
  * @Provides를 사용했을때보다 더 코드가 간단해진다.
  * ```java
    @Binds
    public abstract LoginContract.Presenter
        provideLoginPresenter(LoginPresenter loginPresenter);
  
### BindsOptionalOf(모듈 내 추상메서드 + 매개변수 X)
* 모듈 내 **추상메서드 + 매개변수 X**
* 반환타입필요(void는 안됨)
* 예외사항 못 던짐
* ```java
  @Module
  public abstract class CommonModule {
      @BindsOptionalOf
      abstract String bindsOptionalOfString();
  }
* ```java
  @Module
  public class HelloModule {
      @Provides
      String privdesString() {
          return "Hello";
      }
  }
* ```java
  public class Foo {
      @Inject
      public Optional<String> str;
      
      @Inject
      public Optional<Provider<String>> str2;
      
      @Inject
      public Optional<Lazy<String>> str3;
  }
##### 테스트
* ```java
  @Component(modules = {CommonModule.class, HelloModule.class})
  public interface StrComponent {
      void inject(Foo foo);
  }
* ```java
  @Component(modules = CommonModule.class)
  public interface NoStrComponent {
      void inject(Foo foo);
  }
* ```java
  @Test
  public void testFoo() {
      Foo foo = new Foo();
      
      DaggerStrComponent.create().inject(foo);
      System.out.println(foo.str.isPresent());
      System.out.println(foo.str.get());
      
      DaggerNoStrCpomponent.create().inject(foo);
      System.out.println(foo.str.isPresent());
      System.out.println(foo.str.get());
  }
  
  // true
  // Hello
  // false
  // java.util.NoSuchElementException: No Value present
---
### BindsInstance
* Module을 작성하다 보면 객체 생성시 param으로 입력되는 인자를 외부에서 전달받아야 하는 경우
* Component Builder Setter 또는 Factory 매개변수에 붙일수 있다.
* @Inject가 붙은 필드, 생성자, 메서드에 주입될 수 있다.
* **모듈말고 외부로부터 생성된 인스턴스를 빌더 또느 팩토리를 통해 넘겨줘 해당 인스턴스를 바인딩하게 한다.**
##### Component
* ```java
  @Component(modules = {Module_A.class})
  interface Component_A {

      @Named("Person_Master")
      PersonA callPerson();

      @Component.Builder
      interface Builder {
          @BindsInstance
          Builder setAge(int age);
          Component_A build();
      }
  }
##### Module
* 외부에서 받을 값은 @Named 와 같은 annotation을 설정해주지 않는다.
* ```java
  @Module
  public class Module_A {

      @Provides
      @Named("Name")
      String module_Provides_String_Name(){
          return "sudeky";
      }

      @Provides
      @Named("Age")
      int module_Provides_Int_Age(){
        r eturn 29;
      }

      @Provides
      @Named("Person_Master")
      PersonA module_Provides_PersonA_Person(@Named("Name") String name, int age){ // name은 annotation 설정 안함
          return new PersonA(name, age);
      }

  }
##### PersonA
* ```java
  public class PersonA {

    String name;
    int age;

    @Inject
    public PersonA(String name, int age) {
        this.name = name;
        this.age = age;
    }
  }
##### 실제 Dagger 클래스
* setAge로 인자를 넣어주는것을 확인할수 있다.
* DaggerComponent_A.class
  ```java
  ..
  ...
  @Override
  public PersonA callPerson() {
    return Module_A_Module_Provides_PersonA_PersonFactory.module_Provides_PersonA_Person(module_A, Module_A_Module_Provides_String_NameFactory.module_Provides_String_Name(module_A), setAge);}

  private static final class Builder implements Component_A.Builder {
    private Integer setAge;

    @Override
    public Builder setAge(int age) {
      this.setAge = Preconditions.checkNotNull(age);
      return this;
  }
  ..
  .
##### @Test
* ```java
  @Test
  ...
  Component_A component = DaggerComponent_A.builder()
                .setAge(30)
                .build();
---             
### @Multibinds
* @IntoSet, @ElementsIntoSet 또는 @IntoMap 바인딩이 하나 이상있는 세트 또는 맵이 있으면 상관없지만 만약 비어있을경우 @Multibinds를 사용해야 한다.
  * ```kotlin
    @Module
    abstract class MyModule {
        @Multibinds abstract Set<Foo> aSet();
        @Multibinds @MyQualifier abstract Set<Foo> aQualifiedSet();
        @Multibinds abstract Map<String, Foo> aMap();
        @Multibinds @MyQualifier abstract Map<String, Foo> aQualifiedMap();
    }
* 추상적 멀티바인딩에 사용
* 반환형이 Map 혹은 Set이어야 사용 가능(예 : abstract fun strings() :Set<String>)
* ```kotlin
  @Multibinds
  abstract fun bindsViewModels(): Map<Class<out ViewModel>, @JvmSuppressWildcards ViewModel>
