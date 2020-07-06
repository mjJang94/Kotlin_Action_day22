# 수신 객체 지정 람다: with와 apply
이 두 함수는 매우 편리하며 많은 사람들이 보편적으로 사용한다.   
기능은 바로 수신 객체를 명시하지 않고 람다의 본문 안에서 다른 객체의 메소드를 호출할 수 있게 한다. 그런 람다를 수신 객체 지정 람다라고 부른다.   
먼저 with 함수를 살펴보면 with는 수신 객체 지정 람다를 활용한다.   

## with 함수
특정 객체의 이름을 반복하지 않고도 그 객체에 대해 다양한 연산을 수행할 수 있으면 좋을것이다.   
코틀린도 그러한 기능을 제공하지만 언어 구성 요소로서 제공하지는 않고 with라는 라이브러리 함수를 통해 제공한다.

<pre>
<code>
fun alphabet(): String {
  val result = StringBuilder()
  for(letter in 'A'..'Z') {
    result.append(letter)
   }
   result.append("\nNow I know the alphabet!")
   return result.toString()
}

>>> println(alphabet())
ABCDEF....WXYZ
Now I know the alphabet!
</code>
</pre>

위 코드를 보면 result에 대해 다른 여러 메소드를 호출하면서 매번 result를 반복적으로 사용하고 있다. 이 코드정도 반복은 괜찮은데, 코드가 엄청 길어지거나 result가 엄청많이 불린다면 분명 보기 좋지 않은 코드가 될 것 같다. 이제 with를 사용하여 바꿔보자.

<pre>
<code>
fun alphabet(): String{
  val stringBuilder = StringBuilder()
  return with(stringbuilder) { //----메소드를 호출하려는 수신 객체를 지정
    for(letter in 'A'..'Z') {
      this.append(letter) //----this를 명시하여 앞에서 지정한 수신 객체의 메소드를 호출
    }
  append("\nNow I know the alphabet!")
  this.toString()
</code>
</pre>

with문을 들여다보면 실제로는 파라미터를 2개 갖는 함수이다. 첫번째 파람은 stringBuilder이고, 두번째 파람은 람다이다.   
람다를 괄호 밖으로 빼내는 관례를 사용함에 따라 전체 함수 호출이 언어가 제공하는 특별 구문처럼 보인다. 물론 이 방식 대신 with(stringBulder, {...})라고 쓸 수 있지만 지저분해진다.   
with 함수는 첫 번째 인자로 받은 객체를 두 번째 인자로 받은 람다의 수신 객체로 만든다. 인자로 받은 람다 본문에서는 this를 사용하여 그 수신 객체에 접근할 수 있다.

위 코드에서 this는 with의 첫 번째 인자로 전달된 stringBuilder다. stringBuilder의 메소드를 this.append(letter) 처럼 this 참조를 사용하여 접근하거나 append처럼 바로 호출할 수 있다.

with의 생김새
<pre>
<code>
inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}
</code>
</pre>

다른 예제의 with
<pre>
<code>
class Person {
    var name: String? = null
    var age: Int? = null
}
val person: Person = getPerson()
print(person.name)
print(person.age)
</code>
</pre>
to
<pre>
<code>
val person: Person = getPerson()
with(person) {
    print(name)
    print(age)
}
</code>
</pre>

# apply 함수
apply함수는 with와 거의 같다. 유일한 차이점은 apply는 항상 자신에게 전달된 객체를 반환한다는 점 뿐이다.
<pre>
<code>
fun alphabet() = Stringbulder().apply {
  for(letter in 'A'..'Z'){
    append(letter)
   }
   
   append("\nNow I know the alphabet!")
}.toString()
</code>
</pre>

apply는 확장 함수로 수신 객체가 전달받은 람다의 수신객체가 된다.   
이 함수에서는 apply를 실행한 결과는 StringBuilder 객체이다. 따라서 그 객체의 toString을 호출해서 String타입의 객체를 얻을 수 있다.   
이런 apply 함수는 객체의 인스턴스를 만들면서 즉시 프로퍼티 중 일부를 초기화해야 하는 경우 유용하다. 자바에서는 보통 별도의 Builder 객체가 이런 역할을 담당한다.   
apply를 객체 초기화에 활용하는 예로 안드로이드의 TextView 컴포넌트를 만들면서 특성 중 일부를 변경해보자.

<pre>
<code>
fun createWuthCustomAttributes(context: Context) = 
  TextView(context).apply{
    text = "Sample Text"
    textSize = 20.0
    setPadding(10, 0, 0, 0)
  }
</code>
</pre>

이렇게 간결하게 사용할 수 있는데, 새로운 TextView인스턴스를 만들고 즉시 그 인스턴스를 apply에 넘긴다. apply에 전달된 람다 안에서는 TextView가 수신 객체가 된다.   
람다를 실행하고 나면 apply는 람다에 의해 초기화된 TextView 인스턴스를 반환하고 그 인스턴스는 함수의 결과가 된다.
