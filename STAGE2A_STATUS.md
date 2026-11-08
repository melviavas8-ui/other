# Daamky 1.0 — Этап 2, контрольная точка A (ядро Aura перенесено, НЕ собрано)

Архив `daamky_zenith_core_stage2a.tar.gz` содержит только НОВЫЕ файлы (130 .java + ru.language).
Ни один существующий файл Daamky не изменён. Распаковывать в корень проекта (`src/...`).

## Что сделано
- Из Zenith перенесено ядро Aura: 126 классов в `daamky.zenith.*` (ротации, события, настройки, AttackUtils/AuraUtil/PlayerSimulation, TaskScheduler, утилиты) + AutoTotem/AutoMace/AutoMine/Criticals/AntiBot/WallBypass/ElytraHelper/InventorySetting — их Aura использует сама.
- Логика Aura не переписывалась. Правки только в местах стыка:
  - `AuraModule`: убраны обращения к ReachV3 и CrystalAura (оба в списке исключений) — условия заменены на `false`.
  - `AutoTotemModule`: убраны обращения к CrystalAura; статическая проверка цели вставлена в код.
  - `AuraUtil`, `EntityESP` (не в этом срезе): проверка «питомца» (косметика) заменена на `false`.
  - `Module`: убраны premium-проверка, HolyWorld-трекер, превью модуля.
  - `KeyBindUtils`: убраны проверки экранов Zenith/ботов.
  - `ViaCompat`: убрана зависимость от ViaFabricPlus.
  - `ChatLogger`: фиксированная сине-чёрная палитра Daamky вместо Zenith StyleManager.
  - Удалены артефакты декомпиляции (Lombok `@Generated`, Kotlin-импорты).
- Новые файлы-заглушки: `Zenith` (минимальный хаб: ротации, планировщик, локализация, друзья, детектор сервера), `shim/StyleStub`, `shim/Notifier`, библиотека событий `com.darkmagician6.eventapi` (своя реализация: register/unregister/call, @EventTarget, приоритеты).
- Не перенесено по твоим правилам: CrystalAura, AutoTrap, Blink, FakeLag, Reach, ReachV3, GrimGlide, WaterRise, BoatFly, ShaderESP, BaseFinder, косметика/Wardrobe/питомцы, экраны, HUD, боты, скрипты, Discord, облако/лицензии Zenith.
- Статическая проверка: все внутренние ссылки между 130 файлами разрешаются, пакеты совпадают с путями, скобки сбалансированы. Компилятора здесь нет — это НЕ проверка сборки.

## Что ещё НЕ сделано (без этого Aura не заработает)
1. **Миксины** (самое важное). Нужны 7 хуков Zenith, которые порождают события для Aura: MixinClientPlayerEntity (Update/PostMotion/SprintUpdate/PlayerMove), MixinKeyboardInput (MoveInput, плюс реализация `ViaCompat.Handler`), MixinClientConnection (SendPacket), MixinClientWorld (EntityAdd), MixinMinecraftClient (HandleInput/Tick/Event9), MixinPlayerEntity (Attack), MixinClientPlayNetworkHandler (HealthUpdate). В оригинале они завязаны на модули вне среза (NoInteract, FastBreak, AutoSprint…) — их нужно аккуратно урезать, а mixin-json написать заново. Свой `ClientPlayerEntityMixin` + `Rot` придётся объединить с хуками ротаций Zenith, чтобы не было двух хуков на `sendMovementPackets`.
2. **Адаптер модулей/настроек**: обёртка Zenith Module → `daamky.core.Module` (имя, описание, категория, включение, бинд) и Zenith-настройки → ваши Bool/Mode/Slider/Group (MultiSelect → группа из Bool). Сохранение в ваш ConfigManager.
3. **Регистрация**: замена `AttackAura` на Zenith Aura в ModuleManager (старый файл удалить только после успешной сборки), `Zenith.getInstance()` вызвать в `DaamkyClient` до загрузки модулей.
4. **Gradle/Actions**: Lombok и Kotlin больше не нужны; зависимостей, кроме Fabric API, не требуется. Нужен реальный лог `gradle build` — здесь запустить сборку нельзя (нет доступа к Maven).

## Риски
- Имена методов Yarn 1.21.11 и сигнатуры для миксинов проверить без компилятора нельзя; ошибки вылезут при первой сборке.
- Часть сигнатур Zenith обфусцирована (`method####`), ручные правки только по месту.
