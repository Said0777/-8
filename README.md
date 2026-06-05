# Clean Architecture + BLoC — Flutter App

> Flutter-приложение построенное на принципах **Clean Architecture** с использованием **BLoC** как state manager.

---

## Архитектура

Проект разделён на 3 независимых слоя. Зависимости направлены строго внутрь: `presentation → domain ← data`.

```
lib/
├── core/
│   └── error/
│       └── failures.dart               # Классы ошибок
│
├── data/                               # Слой данных
│   ├── datasources/
│   │   └── post_remote_datasource.dart # HTTP-запросы к API
│   ├── models/
│   │   └── post_model.dart             # Модель с fromJson
│   └── repositories/
│       └── post_repository_impl.dart   # Реализация репозитория
│
├── domain/                             # Бизнес-логика (чистый Dart)
│   ├── entities/
│   │   └── post.dart                   # Сущность без зависимостей
│   ├── repositories/
│   │   └── post_repository.dart        # Абстракция (интерфейс)
│   └── usecases/
│       └── get_posts.dart              # Use Case
│
├── presentation/                       # UI слой
│   ├── bloc/
│   │   ├── post_bloc.dart              # BLoC
│   │   ├── post_event.dart             # События
│   │   └── post_state.dart             # Состояния
│   └── pages/
│       ├── home_page.dart              # Главный экран
│       └── detail_page.dart            # Экран деталей
│
├── injection_container.dart            # Dependency Injection (get_it)
└── main.dart
```

---

## Зависимости

```yaml
flutter_bloc: ^8.1.3   # State management
http: ^1.1.0           # HTTP-запросы
equatable: ^2.0.5      # Сравнение объектов
get_it: ^7.6.4         # Dependency Injection
```

---

## Запуск

```bash
# Установить зависимости
flutter pub get

# Запустить приложение
flutter run
```

---

## Функционал

| Действие | Событие | Состояние |
|---|---|---|
| Открытие экрана | `LoadPostsEvent` | `PostLoadingState` → `PostLoadedState` |
| Перезагрузка (FAB) | `ReloadPostsEvent` | `PostDeleteLoadingState` → `PostLoadedState` |
| Удаление поста | `DeletePostEvent(index)` | `PostDeleteLoadingState` → `PostLoadedState` |
| Переход на детали | `NavigateToDetailEvent(post)` | `PostNavigateToDetailState` |

---

## BLoC: События и Состояния

**События (`post_event.dart`)**
```dart
abstract class PostEvent {}

class LoadPostsEvent extends PostEvent {}
class ReloadPostsEvent extends PostEvent {}
class DeletePostEvent extends PostEvent { final int index; }
class NavigateToDetailEvent extends PostEvent { final Post post; }
```

**Состояния (`post_state.dart`)**
```dart
abstract class PostState {}

class PostInitialState extends PostState {}
class PostLoadingState extends PostState {}
class PostLoadedState extends PostState { final List<Post> posts; }
class PostDeleteLoadingState extends PostLoadedState {}
class PostErrorState extends PostState { final String message; }
class PostNavigateToDetailState extends PostLoadedState { final Post selectedPost; }
```

---

## Слои архитектуры

### Domain — бизнес-логика
Чистый Dart, без зависимостей от Flutter или внешних пакетов.
- **Entity** `Post` — базовая сущность
- **Repository** `PostRepository` — абстрактный интерфейс
- **UseCase** `GetPosts` — единица бизнес-логики

### Data — работа с данными
Реализует контракты domain-слоя.
- **DataSource** — HTTP-запросы к `jsonplaceholder.typicode.com`
- **Model** `PostModel` — extends `Post`, добавляет `fromJson`
- **RepositoryImpl** — реализует `PostRepository`

### Presentation — UI
- **BlocProvider** оборачивает экран и создаёт экземпляр блока
- **BlocConsumer** используется для навигации через `listener`
- **BlocBuilder** строит UI в зависимости от текущего состояния

---

## Dependency Injection

Регистрация зависимостей через `get_it`:

```dart
void init() {
  sl.registerFactory(() => PostBloc(getPosts: sl()));
  sl.registerLazySingleton(() => GetPosts(sl()));
  sl.registerLazySingleton<PostRepository>(() => PostRepositoryImpl(sl()));
  sl.registerLazySingleton<PostRemoteDataSource>(() => PostRemoteDataSourceImpl(sl()));
  sl.registerLazySingleton(() => http.Client());
}
```

---

## API

Данные загружаются с публичного REST API:

```
GET https://jsonplaceholder.typicode.com/posts
```
