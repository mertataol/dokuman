# Jenerikler

Bir veri türü oluşturulurken yahut bir işlev tanımlanırken bunların farklı türde argümanlarla da çalışması istenir. Rust' ta **jenerikler**, veri türlerini tek noktada toplayarak kodun başka türler için tekrar yazılmasını önler. Farklı veri türleri için  genelleştirilmiş olan algoritmanın, her veri türü için tekrar üretilmesi gerekmeyeceğinden, programın kod tasarımı sadeleşmiş geliştirme hızı da artmış olur. 

Genelleştirme kavramında özel bir veri örn: `(x: u8)` türü bildirmek yerine türün yerine geçebilen örn: `(x: T )` gibi genel bir belirteç kullanılır. Ancak genel türün derleyici tarafından anlaşılabilmesi için `<T>` şeklinde tanımlanarak bildirilmesi gerekmektedir.

### Jenerik işlevler
Aynı işlevin farklı türlerle kullanılabiliyor olması kodun gereksizce uzamasını önleyerek daha esnek olmasını sağlar.
```Rust
fn her_ture_uygun<T>(x: T) { 
    // x T türünde bir parametredir. T ise jenerik türdür yani farklı türleri için genelleştirilmiştir.  
} 

fn ayni_turden_iki_tane<T>(x: T, y: T) { 
    // Her ikisi de aynı türden parametre bekler 
} 

fn farkli_turden_iki_tane<T, U>(x: T, y: U) {  
    // Farklı türde parametreler.
}
````
Bir verinin hangi tür olduğunu öğrenebilmek için `std::any` kütüphanesinden yararlanabiliriz.
```Rust
fn her_ture_uygun<T>(_: T) { 
    // x T türündedir. T ise jenerik türdür yani farklı türleri için genelleştirilmiştir.  
    println!("Bu veri {} türündedir", std::any::type_name::<T>());
} 

fn main() {
    
    let bir_tur = 6;
    //let bir_tur = 65u8;
    // let bir_tur = String::from("Merhaba");
    her_ture_uygun(bir_tur);
}
````

### Jenerik yapılar
Jenerik tür parametrelerinin yapı alanlarında kullanılabilmesi için tanımlarında `<T>` söz diziminin kullanılması gereklidir. Herhangi bir türden oluşan `x` ve `y` kordinatlarını tutan `Nokta<T>` yapısı aşağıda örneklenmiştir.

```Rust
struct Nokta<T> {
    x: T,
    y: T,
}

fn main() {
    let tamsayi = Nokta{x: 5, y: 10};
    let kesirli = Nokta{x: 3.5, y: 9.2};
    
    println!("Tam sayı için koord: {} - {}", tamsayi.x, tamsayi.y); // Tam sayı için koord: 5 - 10
    println!("Kesirli için koord: {} - {}", kesirli.x, kesirli.y);  // Kesirli için koord: 3.5 - 9.2
}
````
Jenerik işlevlerde olduğu gibi; yapı tanımında bildirilen tür parametresi `<T>`' nin bir kez kullanılması, yapının tüm alanlarının aynı türden oluşacağını gösterir. `let tamsayi = Nokta{x: 5, y: 10.7};` şeklinde oluşturulan bir yapı örneği bu programın hata üretmesine sebep olacaktır. 

Farklı türden alanlara sahip bir yapıya ihtiyaç duyulduğunda, bu türlerin yapı tanımında bildirilmesi yeterlidir. Ancak yapı tanımında çok sayıda tür parametresinin kullanılması kodun okunmasını zorlaştırır. Bir yapı tanımında çok sayıda genel türe ihtiyaç duyuluyorsa belki de kodun küçük parçalar halinde yeniden tasarlanması fikri üzerinde düşünülmelidir.    
```Rust
struct Nokta<T, U> {
    x: T,
    y: U,
}

fn main() {
    let tamsayi = Nokta{x: 5, y: 10};
    let kesirli = Nokta{x: 3.5, y: 9.2};
    let karisik = Nokta{x: 7, y: 3.2};
    
    println!("Tam sayı için koord: {} - {}", tamsayi.x, tamsayi.y); // Tam sayı için koord: 5 - 10
    println!("Kesirli için koord: {} - {}", kesirli.x, kesirli.y);  // Kesirli için koord: 3.5 - 9.2
    println!("Karisik için koord: {} - {}", karisik.x, karisik.y);  // Karisik için koord: 7 - 3.2
}
````

Jenerik yapılar için uygulama eklenirken tür parametreleri `impl` anahtar kelimesinden sonra belirtilmelidir.
```Rust
struct Nokta<T, U> {
    x: T,
    y: U,
}
// Parantez gösterimi
impl<T, U> Nokta<T, U> {
    fn yeni(x: T, y: U) -> Self {
        Self {
            x,
            y,
        }
    }
    
    fn degistir<V, W>(self, oteki: Nokta<V, W>) -> Nokta<T, W> {
        Nokta {
            x: self.x,
            y: oteki.y,
        }
    }
} 

fn main() {
    let tamsayi = Nokta::yeni(5, 7);
    println!("{} - {}", tamsayi.x, tamsayi.y);
    
    let dizge   = Nokta::yeni("Merhaba", 'p');
    println!("{} - {}", dizge.x, dizge.y);
    
    let donustur   = tamsayi.degistir(dizge);
    println!("{} - {}", donustur.x, donustur.y);
}
````
### Jenerik enum
Yapılarda olduğu gibi jenerik veri türlerini varyantlarında tutabilen enum türlerinden de yararlanabiliriz. Rust standart kitaplığında daha önceden tanımlanmış özel türlerden Option<T> ve Result<T> türleri bu konuya oldukça iyi birer örnektir.
```Rust
enum Option<T> {
    Some(T),
    None,
}

enum Result<T, E> { 
    Ok(T), 
    Err(E), 
}
````
İsteğe bağlı bir `Some` değerine sahip olan  türü soyut kavramları ifade etmekte oldukça yararlıdır. İsteğe bağlı değerin türü ne olursa olsun, `Option<T>` genel bir türü ifade ettiğinden bu soyutlama pekçok veri türüyle kullanılır.  

Duruma göre ya başarılı `Ok` ya da başarısız `Err` değer döndüren Result<T, E> ise iki genel türden oluşur. 

Bir Result türü ise  olabilir. 