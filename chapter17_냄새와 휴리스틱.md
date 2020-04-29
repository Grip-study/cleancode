## G2 : 당연한 동작을 구현하지 않는다.

```kotlin
class 계산기 {
    fun sum(vararg value: Int) : Int{
        ... // 적당히 모든 값을 더해서 리턴
    }
    
    fun minus(first : Int , second: Int) : Int{
        ... // 적당히 빼서 리턴
    }
    
    fun mul(vararg value: Int) : Int{
        ... // 적당히 모든 값을 곱해서 리턴
    }
    
    fun div(first : Int , second: int) : Int{
        ... // 적당히 나눠서 값을 리턴
    }
}
```

-> 나누기가 인트형으로 나와서 매우 실망스러움

---

## G3 : 경계를 올바로 처리하지 않는다.
```kotlin
class MainActivity {
    ...
    fun initView(){
        ... 
        Glide.with(this).load(이미지URL).into(imageView)
    }
}
```
-> Glide는 Url로 이미지뷰에 이미지를 그려줄 수 있는 외부라이브러리.

-> 버전업을 했더니 사용방법이 바뀜
```kotlin
class MainActivity {
    ...
    fun initView(){
        ... 
        Gliiide.wiiith(this).iiinto(imageView).load(이미지URL)
    }
}
```
-> 결국 기존의 Glide를 사용하던곳 다 찾아다니며 바꿔줘야함.

```kotlin
class ImageLoader {
    ...
    fun loadImage(imgUrl: String, view: ImageView){
        Gliiide.wiiith(this).iiinto(imageView).load(이미지URL)
    }
}
```
-> 외부 라이브러리를 감싸는 클래스를 만들어서 사용하면 고치기 쉽고 테스트하기에도 용이

---

## G4 : 안전 절차 무시

-> 기존의 코드
```kotlin
class University(올해년도 : Int) {
    val students = listOf<String>()
    val subjects = HashMap<String, List<String>>()
    
    ...
    
    fun 수강신청(subjectName: String, student: Student){
        //적당히 subject에 잘 넣는 코드
    }
    
    fun 나와라수강생들(subjectName: String): List<String>{
        //적당히 subjects 학생리스트 꺼내오는 코드
    }
    
    fun 결석한사람수(subjectName: String, student: List<String>): Int{
        //적당히 결석한 사람 수 리턴하는 코드
    }
}

class Main{
    fun main(vararg arg: String){
        ....
        university.수강신청("전쟁과평화", "오승열")
        university.수강신청("전쟁과평화", "육승구")
        university.수강신청("전쟁과평화", "칠승팔")
        
        ...
        
        val students = university.나와라수강생들("전쟁과평화")
        ㅌㅌ(student, "육승구")
        
        ...
        
        val ㅌㅌㅌ = 결석한사람수("전쟁과평화", students)
    }
    
    fun ㅌㅌ(students: List<String>, student: String){
        students.remove(student)
    }
}
```

-> 아무생각없이 고친 코드
```kotlin
class University(올해년도 : Int) {
    val students = mutableListOf<String>()
    val subjects = HashMap<String, List<String>>()
    
    ...
    
    fun 수강신청(subjectName: String, studentName: String){
        //적당히 subject에 잘 넣는 코드
    }
    
    fun 나와라수강생들(subjectName: String): List<String>{
        //적당히 subject에서 학생리스트 꺼내오는 코드
    }
    
    fun 자퇴(studentName: String){
        students.remove(studentName)
    }
}

class Main{
    fun main(vararg arg: String){
        ....
        university.수강신청("전쟁과평화", "오승열")
        university.수강신청("전쟁과평화", "육승구")
        university.수강신청("전쟁과평화", "칠승팔")
        
        ...
        
        university.자퇴("오승열")
        
        ...
        
        val students = university.나와라수강생들("전쟁과평화")
        ㅌㅌ(student, "육승구")
        
        ...
        
        val ㅌㅌㅌ = 결석한사람수("전쟁과평화", students)
    }
}
```

-> 자퇴를 별 생각없이 학생리스트에서 지우면 끝이라 생각해 지웠지만

subjects에 그대로 남아 있어 버그가 생겨버렸다.
