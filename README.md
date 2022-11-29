## Refactoring 2nd - 9장 데이터 조직화

### 9장을 공부하며 Rust를 사용한 이유

  * 메모리 안전성/자유성
  
    - 스택과 힙을 넘나들며 객체를 만들 수 있습니다.
    - 쓰레기 수집기(GC)가 없기에 내가 사용중인 객체에 대한 수명을 생각할 힘을 길러줍니다.
    - 각종 생성자들을 직접 구현해야 하나, 이를 통해 얕은 복사, 깊은 복사가 일어날 상황을 명확히 구분할 수 있습니다.
  
  * mutable/Immutable, 참조/비참조의 엄격함
  
    - let mut 변수 -> Mutable, let 변수 ->Immutable으로써, 수정 가능/불가능한 변수를 엄격히 나누어 생각할 힘을 길러줍니다.
    - Rust에서는 메서드 호출시 개체를 참조할지 소유권을 이전할지 조차 매개 변수로 지정합니다.

  * 그래서 책의 자바스크립트 보다 훠얼씬 불편하지만, 그 불편함을 통해 배울 것이 많은 Rust로 구현해봤습니다. 
  
### 9.1 임시 변수 분리

  * 기본 예제
  
    - 다음 변수를 쪼개봅시다!
    
      ``` Rust
      fn print_info(height : u32, width: u32)
      {
          let mut temp = 2 *(height + width);
          
          print!(temp);
          
          temp = height * width;  
          
          print!(temp);
      }
      ```
      
    - Immutable(변경 불가능한, const 변수 같은 거) 변수로 나눕시다!
      ``` Rust
      fn print_info(height : u32, width: u32)
      {
          let length = 2 *(height + width); //길이 변수랑
        
          print!(length);
        
          let area = height * width; //면적 변수로
        
          print!(area);
        
          //temp라는 임시 변수를 나누었습니다! 의미가 명확해진 거 같습니다!
      }
      ```
      
  * 예시: 입력 매개변수의 값을 수정할 때
  
    - 다음 코드를 고쳐봅시다!
      ``` Rust
      fn discount(original_input_val: u32, quantity: u32) -> u32 {
      
          let mut input_value = original_input_val.clone();
          
          if input_value > 50 {
          
             input_value = input_value - 2;
             
          }
          if quantity > 100 {
          
             input_value = input_value - 1;
             
          }
          
           return input_value;
      }
      ```
    - 무슨 동작을 하는지 한눈에 보이기 시작했습니다!
      ``` Rust
      fn discount(input_value: u32, quantity: u32) -> u32 {
      
          let mut result = input_value.clone();
          
          if input_value > 50 {
          
             result = result - 2;
             
          }
          if quantity > 100 {
          
             result = result - 1;
             
          }
          
           return result;
      }
      ```
    

    
### 9.2 필드 이름 바꾸기

   * 예시
   
      - 클래스(구조체)에 마음에 들지 않는 필드 이름이 보입니다!
      
        ``` Rust
        struct organization
        {
          name: String, // 맘에 들지 않으니 title로 바꾸고 싶습니다!
          country: String
        }
      
        impl organization
        {
          fn new(name: String, country: String) ->Self //생성자
          {
            Self
            {
              name: name,
              country: country
            }
          }
          
          fn get_name(&self) -> String //self -> X &self -> O : 메서드 호출시 개체가 함수 안에서 소멸하지 않도록 "참조" 해줍니다!
          {
            self.name
          }
        }
        ```
      - 일단 클래스(구조체) 내부 변수명(필드명)부터 바꾸어줍니다.
      
        ``` Rust
        struct organization
        {
          title: String, // 필드명부터 바꾸고 꼭!! 테스트 합니다!
          country: String
        }
      
        impl organization
        {
          fn new(name: String, country: String) ->Self //생성자
          {
            Self
            {
              title: name,
              country: country
            }
          }
          
          fn get_name(&self) -> String
          {
            self.title
          }
        }
        ```
      - 위 과정에 대한 테스트를 충분히 하고, 외부에서 호출하는 메서드를 수정하세요! 무조건 테스트! 테스트!
      
        ``` Rust
        struct organization
        {
          title: String, // 일단 이건 바꾸었고,
          country: String
        }
      
        impl organization
        {
          fn new(title: String, country: String) ->Self //생성자 매개변수도 수정!
          {
            Self
            {
              title: title,
              country: country
            }
          }
          
          fn get_title(&self) -> String // 매개변수, 함수(메서드) 이름 수정! 
          {
            self.title
          }
        }
        //휴 끝났다..가 아니죠 꼭!! 테스트하세요!!
        ```

### 9.3 파생 변수를 질의 함수로!
   
   * 예시
   
      - 함수의 기능에 주목합시다!   
        ``` Rust
        struct sale_info {
            discounted_total: u32,
            base_total: u32,
            discount: u32,
        }

        impl sale_info {
            fn set_discount(&mut self, number: u32) {
              let old = self.discount;        //하나의 메서드 안에서 파생 변수를 만들어 여러 변수를 반복해서 바꾸고 있습니다.
              self.discount = number;
              self.discounted_total += old - number;
            }
            fn get_discounted_total(&self) -> u32 {
                self.discounted_total
            }     
        }
        ``` 
      - 그래서 내부 파생 변수를 참조해서 반환하는 get_discounted_total() 함수를 바꾸어봅시다!
        ``` Rust
        struct sale_info {
            //discounted_total: u32, //-> 파생해서 나온 변수는 제거합시다!
            discount: u32,
            base_total: u32, // 이 변수를 통해
            //Total Discount는 요청하면 그때그때 계산해서 함수로 반환합시다!
        }

        impl sale_info {
            fn set_discount(&mut self, number: u32) {
              self.discount = number;
            }
            fn get_discounted_total(&self) -> u32 { 
                self.base_total - self.discount //훨씬 깔끔해 보입니다!
            }     
        }
        ``` 
        
### 9.4 참조를 값(혹은 복사된 개체)으로 바꾸기!

   * 예시
    
     - 여기 생성 시점에서 내부 필드(변수값)가 지정되지 않은 전화번호 클래스(구조체)가 있습니다.
     
        ``` Rust
          struct TelephoneNumber {
            area_code: u32,
            number: u32,
          }
          struct Person {
             telephone_number: Rc<TelephoneNumber>, 
             //Rc<타입> : Rust에서 참조 횟수라는 것으로 수명을 정하는 스마트 포인터, Rc입니다.
             //무섭게 생겼으나, 원리는 간단합니다. 힙 할당 후, 참조하면 횟수가 오르고 참조하지 않으면 횟수가 줄어들다가
             //0이 되면, 자기 자신을 할당 해제 합니다.
             // 이것은 Swift라는 언어와 동일한 방식입니다.
          }
        
          impl Person {
           fn new() -> Self {
              Self {
                telephone_number: Rc::new(TelephoneNumber {//본론으로 돌아와서 
                area_code: 0,  //생성자에서 불필요하게 객체가 초기화 되는 모습을 여기서 볼 수 있습니다!
                number: 0 
                }),
                  }
            }
            fn set_office_area_code(&mut self, area_code: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //불필요한 참조가 여기도 있고,(강한 참조 + 1)
              ptr.area_code = area_code;
            }
            fn get_office_area_code(&self) -> u32 {
              self.telephone_number.area_code
            }
            fn set_office_number(&mut self, number: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //여기도 있습니다.(강한 참조 + 1)
              ptr.number = number;
            }
            fn get_office_number(&self) -> u32 {
              self.telephone_number.number
            }
          }
        ```
     
     - 그래서 참조를 제거하고 복사로 깔끔하게 바꾸어 보려고 합니다!
        ``` Rust
         struct TelephoneNumber {
          area_code: u32,
          number: u32,
         }
         impl TelephoneNumber {
          fn new(area_code: u32, number: u32) -> Box<Self> {
           Box::new(Self {
            area_code: area_code,
            number: number, //Box<타입>: Rust를 이루는 모든 것이라할 수 있는 "소유권" 기반 힙 스마트 포인터 입니다.
                            //여기서 설명하려면 머리 아프니 
                            //자바,JS에서 new TelephoneNumber()와 같이 힙에 객체를 할당해서 반환하는 생성자라고 생각해주세요!
          })
          }
         }
         struct Person {
           telephone_number: Box<TelephoneNumber>,
          }
         
          impl Person {
           fn new(area_code: u32, number: u32) -> Self {
            Self {
             telephone_number: TelephoneNumber::new(area_code, number),
            }
           }
           fn set_office_area_code(&mut self, area_code: u32) {
             self.telephone_number = TelephoneNumber::new(area_code, self.telephone_number.number); //new TelephoneNumber(번호, 지역번호); 와 같습니다.
           }
           fn get_office_area_code(&self) -> u32 {
             self.telephone_number.area_code
           }
           fn set_office_number(&mut self, number: u32) {
             self.telephone_number = TelephoneNumber::new(self.telephone_number.area_code, number); //new TelephoneNumber(번호, 지역번호);와 같습니다.
            }
           fn get_office_number(&self) -> u32 {
            self.telephone_number.number
           }
          }
        ```

### 9.5 값(혹은 복사된 개체)를 참조로 바꾸기!

   * 예시
   
     - 불필요한 복사에 대해 거대한 하나의 객체로 부터 참조 받아오는 방법이 책에 소개 되어있습니다.
       ``` Rust
       struct Customer {
          id: String,
       }
       
       impl Clone for Customer {
        fn clone(&self) -> Self { //Customer Class의 복사생성자
         Self {
            id: self.id.clone(),
         }
        }
       }
       impl Customer {
        fn new(id: String) -> Self {
          Self { id: id }
         }
       }
       struct Order {
          number: u32,
          customer: Box<Customer>,
       }

       impl Order {
         fn new(number: u32, id: String) -> Self {
            Self {
               customer: Box::new(Customer::new(id)), //여기서 개체 복사(복사 생성)가 발생해, 참조가 아닌 새로운 값(객체)으로 할당하고 있습니다.
               number: number,
            }
         }
       }
       ```
     - 이러한 경우에는 Map자료형으로 된 거대한 하나(Singleton)의 객체 저장소를 만들어 사용하는 방식을 저자가 추천하고 있습니다!
       ``` Rust
        use std::collections::HashMap;
        use std::rc::*;
        struct Customer {
          id: String,
        }
        impl Clone for Customer {
          fn clone(&self) -> Self {
           Self {
            id: self.id.clone(),
           }
          }
        }
        impl Customer {
         fn new(id: String) -> Self {
          Self { id: id }
            }
        }
        static mut REPOSITORY: Option<HashMap<String, Rc<Customer>>> = None; //하나의 큰(Singleton) 저장소 객체를 전역적으로 구현하기 위함.
        unsafe fn init_repo() {
             REPOSITORY = Some(HashMap::new())
        }
        unsafe fn register_customer(id: String) -> Rc<Customer> {
           if (REPOSITORY.is_none()) {
              init_repo();
           };
           if REPOSITORY.as_mut().unwrap().get(&id.clone()).is_none() {  //책과 살짝 다르지만 동작은 같습니다.
             REPOSITORY
            .as_mut()
            .unwrap()
            .insert(id.clone(), Rc::new(Customer::new(id.clone())));
           }
           return Rc::clone(REPOSITORY.as_mut().unwrap().get(&id.clone()).unwrap()); 
           //참조를 반환하기 위해 참조 기반 스마트 포인터인 RC를 사용했습니다.
        }
        struct Order {
          number: u32,
          customer: Rc<Customer>,
        }
        impl Order {
          fn new(number: u32, id: String) -> Self {
            Self {
                 customer: unsafe {register_customer(id)},  
                 //하나의 큰 저장소의 맵 자료구조에다가 저장해놓고, 그 안에서의 위치(참조)받아서 저장
                 number: number,
               }
            }
        }
       ```
