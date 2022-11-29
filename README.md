# Refactoring 요약본

## 9장 데이터 조직화(Rust 소스 코드로 작성!)

### 9.1 임시 변수 분리

  * 기본 예제
  
    - 다음 코드를 쪼개보자!
    
      ``` Rust
      fn print_info(height : u32, width: u32)
      {
          let mut temp = 2 *(height + width);
          
          print!(temp);
          
          temp = height * width;  
          
          print!(temp);
      }
      ```
      
    - Immutable(변경 불가능한, const 변수 같은 거) 변수로 여러개로 만들어 쪼갠다.
      ``` Rust
      fn print_info(height : u32, width: u32)
      {
          let length = 2 *(height + width); //길이 변수랑
        
          print!(length);
        
          let area = height * width; //면적 변수로
        
          print!(area);
        
          //temp라는 임시 변수를 나누었다
      }
      ```
      
  * 예시: 입력 매개변수의 값을 수정할 때
  
    - 다음 코드를 고쳐보자!
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
    - 이로써 하는 일이 명확해졌다.
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
   
      - 필드이름을 명확히 써보자!
      
        ``` Rust
        struct organization
        {
          name: String, // 맘에 들지 않으니 title로바꾸어보자!
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
          
          fn get_name(&self) -> String
          {
            self.name
          }
        }
        ```
      - 일단 클래스(구조체) 내부 변수부터 바꾸고 나서, 이것을 참조하는 많은 변수들을 수정한다.
      
        ``` Rust
        struct organization
        {
          title: String, // 맘에 들지 않으니 title로바꾸어보자!
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
      - 클래스 내부 변수 이름을 바꾼 후, 그로 생기는 문제를 해결하고 나서 외부에서 호출하는 메서드를 수정하자! 무조건!
      
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
        ```

### 9.3 파생 변수를 질의 함수로!
   
   * 예시
   
      - 함수의 기능에 주목하자!   
        ``` Rust
        struct sale_info {
            discounted_total: u32,
            base_total: u32,
            discount: u32,
        }

        impl sale_info {
            fn set_discount(&mut self, number: u32) {
              let old = self.discount;
              self.discount = number;
              self.discounted_total += old - number;
            }
            fn get_discounted_total(&self) -> u32 {
                self.discounted_total
            }     
        }
        ``` 
      - 내부 변수를 참조해서 반환하던 get_discounted_total() 함수가 요청에 따라 계산하는 함수로 바뀌었다!
        ``` Rust
        struct sale_info {
            //discounted_total: u32, //-> 안녕 잘 가!
            discount: u32,
            base_total: u32, // 이게 더 나은 거 같아!
            //Total Discount는 요청하면 함수로 반환하자!
        }

        impl sale_info {
            fn set_discount(&mut self, number: u32) {
              self.discount = number;
            }
            fn get_discounted_total(&self) -> u32 {
                self.base_total - self.discount
            }     
        }
        ``` 
        
### 9.4 참조를 값(혹은 복사된 개체)으로 바꾸기!

   * 예시
    
     - 생성 시점에서 제대로 지정되지 않은 전화번호 개체를 설정해보자
     
        ``` Rust
          struct TelephoneNumber {
            area_code: u32,
            number: u32,
          }
          struct Person {
             telephone_number: Rc<TelephoneNumber>,
          }
        
          impl Person {
           fn new() -> Self {
              Self {
                telephone_number: Rc::new(TelephoneNumber {
                area_code: 0, 
                number: 0 //이상한 초기값도 넣어줘야 한다.
                }),
                  }
            }
            fn set_office_area_code(&mut self, area_code: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //불필요한 참조 + 1
              ptr.area_code = area_code;
            }
            fn get_office_area_code(&self) -> u32 {
              self.telephone_number.area_code
            }
            fn set_office_number(&mut self, number: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //불필요한 참조 + 1
              ptr.number = number;
            }
            fn get_office_number(&self) -> u32 {
              self.telephone_number.number
            }
          }
        ```
    
    - 아래는 참조를 제거해 RefenceCount(Rc) 방식이 아닌 소유권 힙 할당(Box)를 사용해 깔끔하게 작성되었다.
        ``` Rust
        struct TelephoneNumber {
          area_code: u32,
          number: u32,
        }
        impl TelephoneNumber {
         fn new(area_code: u32, number: u32) -> Box<Self> {
           Box::new(Self {
            area_code: area_code,
            number: number,
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
           let ptr = TelephoneNumber::new(area_code, self.telephone_number.number);
           self.telephone_number = ptr;
         }
         fn get_office_area_code(&self) -> u32 {
           self.telephone_number.area_code
         }
         fn set_office_number(&mut self, number: u32) {
          let ptr = TelephoneNumber::new(self.telephone_number.area_code, number);
          self.telephone_number = ptr;
         }
         fn get_office_number(&self) -> u32 {
            self.telephone_number.number
         }
        }
        ```

### 9.5 값(혹은 복사된 개체)를 참조로 바꾸기!

     - 생성 시점에서 제대로 지정되지 않은 전화번호 개체를 설정해보자
       ``` Rust
          struct TelephoneNumber {
            area_code: u32,
            number: u32,
          }
          struct Person {
             telephone_number: Rc<TelephoneNumber>,
          }
        
          impl Person {
           fn new() -> Self {
              Self {
                telephone_number: Rc::new(TelephoneNumber {
                area_code: 0, 
                number: 0 //이상한 초기값도 넣어줘야 한다.
                }),
                  }
            }
            fn set_office_area_code(&mut self, area_code: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //불필요한 참조 + 1
              ptr.area_code = area_code;
            }
            fn get_office_area_code(&self) -> u32 {
              self.telephone_number.area_code
            }
            fn set_office_number(&mut self, number: u32) {
              let ptr = Rc::get_mut(&mut self.telephone_number).unwrap(); //불필요한 참조 + 1
              ptr.number = number;
            }
            fn get_office_number(&self) -> u32 {
              self.telephone_number.number
            }
          }
       ```
