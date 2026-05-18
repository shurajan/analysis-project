FiX #1
1. Изменил сигнатуру на fn parse<'a>(&self, input:&'a str)->Result<(&'a str, Self::Dest), ParseError>;
   Избавился от лишних String::clone() и .to_string() 
2. Добавлен ParseError с конкретными вариантами типами ошибок
3. stdp::U32 — переход на tight-тип std::num::NonZeroU32

Было: type Dest = u32, с явной проверкой if value == 0 { return Err(ParseError) }.

Стало: type Dest = std::num::NonZeroU32. Проверка на ноль теперь выполняется через систему типов:
let value = std::num::NonZeroU32::new(raw).ok_or(ParseError::ZeroValue)?;
Это устраняет if-ветку: невозможность ноля закодирована в типе.

4. Там, где результат stdp::U32 использовался как u32 (Backet, UserCash, AppLogJournalKind::CreateUser/RegisterAsset, LogLine), в типовой аннотации fn((String, u32))->Self заменено на fn((String, NonZeroU32))->Self, а в теле функции
добавлено .get() при заполнении поля структуры (поля структур остались u32).

4. Обновлены тесты

Все Err(ParseError) заменены на конкретные варианты (Err(ParseError::InvalidNumber), Err(ParseError::ExpectedTag) и т.д.). Возвращаемые значения stdp::U32 обёрнуты в NonZeroU32 через вспомогательную функцию fn nzu(n: u32) ->
NonZeroU32.


FIX #2
1. pub struct AuthData(Box<[u8; AUTHDATA_SIZE]>) — 1024 байта теперь хранятся на куче; сама структура на стеке занимает лишь указатель (8 байт). Заодно исправлен вариант перечисления AppLogTraceKind::Connect, который раньше тащил
   1024 байта на стек при каждом обращении к AppLogTraceKind.
2. Тело парсера — результат обёрнут в Box::new(...). Массив собирается из Vec<u8> через try_into() 
3. Тест — конструкция AuthData(Box::new([...])) обновлена в соответствии с новым типом.