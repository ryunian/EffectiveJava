# 생성자에 매개변수가 많다면 빌더를 고려하라
> 생성자에 매개변수가 많다면 어떻게 할까?  
> 점층적 생성자패턴, 자바빈즈패턴, 빌더패턴을 고려해볼수 있지만, 빌더패턴을 사용하는 이유에 대해 알아보자 

### 1. 점층적 생성자 패턴 (relescoping constructor pattern)
> 점층적 생성자 패턴이 안된다는것은 아니지만, 
> 매개변수 개수가 많아지면 클라이언트 코드를 작성하거나 읽기 어렵다.

```java
public class NutritionFacts {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public NutritionFacts(int servingSize, int servings) {
        this(servingSize, servings, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories) {
        // 생략
        this(servingSize, servings, calories, 0, 0, 0);
    }

    public NutritionFacts(int servingSize, int servings, int calories, int fat, int sodium, int carbohydrate) {
        this.servingSize = servingSize;
        this.servings = servings;
        this.calories = calories;
        this.fat = fat;
        this.sodium = sodium;
        this.carbohydrate = carbohydrate;
    }
}
```

<br><br><br>

### 2. 자바빈즈패턴 (JavaBeans pattern)
> 자바빈즈패턴에서는 객체 하나를 만들려면 메서드를 여러 개 호출해야 되고,
> 객체가 완전히 생성되기 전까지는 일관성이 무너진 상태에 놓이기 된다.

* 일관성이 깨진 객체가 만들어지면, 버그를 심은 코드와 그 버그 때문에   
런타임에 문제를 격는 코드가 물리적으로 멀리 떨어져 있을것이므로 디버깅이 힘들다
* 이 처럼 일관성이 무너지는 문제 때문에 자바빈즈 패턴에서는 클래스를 불변(item 17)으로 만들 수 없다.
```java
package item2;

public class NutritionFacts2 {
    private int servingSize;
    private int servings;
    private int calories;
    private int fat;
    private int sodium;
    private int carbohydrate;

    public NutritionFacts2() {}

    public void setServingSize(int servingSize) {this.servingSize = servingSize;}

    public void setServings(int servings) {this.servings = servings;}

    public void setCalories(int calories) {this.calories = calories;}

    public void setFat(int fat) {this.fat = fat;}

    public void setSodium(int sodium) {this.sodium = sodium;}

    public void setCarbohydrate(int carbohydrate) {this.carbohydrate = carbohydrate;}
}

```

<br><br>

### 3. 빌더 패턴 (Builder pattern)
> 클라이언트는 필요한 객체를 직접 만드는 대신, 필수 매개변수만으로 생성자(혹은 정적 팩터리)를 호출해 빌더 객체를 얻는다.  
> 빌더 객체가 제공하는 일종의 세터 메서드들로 원하는 선택 매개변수들을 설정한다.  
> 마지막으로 매개변수가 없는 build 메서드를 호출해 객체를 얻는다.

> 빌더는 생성할 클래스 안에 정적 멤버 클래스로 만들어두는 게 보통이다.
```java
package item2;

public class NutritionFacts3 {
    private final int servingSize;
    private final int servings;
    private final int calories;
    private final int fat;
    private final int sodium;
    private final int carbohydrate;

    public static class Builder {
        private final int servingSize;
        private final int servings;

        private int calories;
        private int fat;
        private int sodium;
        private int carbohydrate;

        public Builder(int servingSize, int servings) {
            this.servingSize = servingSize;
            this.servings = servings;
        }

        public Builder calories(int val) {
            calories = val;
            return this;
        }

        public Builder fat(int val) {
            fat = val;
            return this;
        }

        public Builder sodium(int val) {
            sodium = val;
            return this;
        }

        public Builder carbohydrate(int val) {
            carbohydrate = val;
            return this;
        }

        public NutritionFacts3 build(){
            return new NutritionFacts3(this);
        }
    }
    private NutritionFacts3(Builder builder) {
        servingSize = builder.servingSize;
        servings = builder.servings;
        calories = builder.calories;
        fat = builder.fat;
        sodium = builder.sodium;
        carbohydrate = builder.carbohydrate;
    }
}
```

* 잘못된 매개변수를 찾기 위해선, 빌더의 생성자와 메서드에서 입력 매개변수를 검사하고,
* build 메서드가 호출하는 생성자에서 여러 매개변수에 걸친 불변식(invariant)을 검사해야 한다.
* 검사해서 잘못된 점을 발견하면 어떤 매개변수가 잘못되었는지를 나타내주는 메시지를 담아 IllegalArgumentException을 던지면된다.

<br><br>

장점
* 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
* 가변인수 매개변수를 여러개 사용할 수 있다.
* 빌더 하나로 여러 객체를 순회하면서 만들수 있다.
* 빌더에 넘기는 매개변수에 따라 다른 객체를 만들수 있다.
* 객체마다 부여되는 일련번호와 같은 특정 필드는 빌더가 알아서 채우도록 할 수도 있다.

<br><br>

단점
* 객체를 생성하기 위해서는 빌더부터 만들어야 한다.  
빌더 생성 비용은 크지않지만 성능상 문제가 될 수 있다. 
* 점층적 생성자 패턴보다는 코드가 장황하다. (4개 이상 되어야 값어치를 한다)

<br><br>

정리
* 생성자나 정적 팩터리가 처리해야 할 매개변수가 많다면 빌더 패턴을 선택하는게 낫다
* 매개변수 중 다수가 필수가 아니거나 같은 타입이면 더 그렇다.
* 빌더는 점층적 생성자보다 클라이언트 코드를 읽고 쓰기가 간결하고, 자바빈즈보다 훨씬 안전하다.