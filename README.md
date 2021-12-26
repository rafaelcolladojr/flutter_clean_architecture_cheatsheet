
# Flutter Clean Architecture Cheatsheet

### The following document serves as a guideline to starting and "bootstrapping" any Flutter project employing the Clean Architecture folder/file structure.


## Folder Struture
##### Funny enough, the folder's structure logical order is also alphabetical. :p
###### (Feel free to move your state management system [I chose bloc] to your most preferred folder/location)


- core
    - error
    - network
- config
- data
    - datasource
    - model
    - repository
- domain
    - repository
    - entity
    - usecase
- presentation
    - bloc
    - page
    - widget

## File Templates

### core/error
##### To be implemented by all Failure types; returned by usecases.
##### `failure.dart`
```
abstract class Failure extends Equatable {
  const Failure();
}

class ServerFailure implements Failure {
  @override
  List<Object?> get props => [props];

  @override
  bool? get stringify => true;
}

class CacheFailure implements Failure {
  @override
  List<Object?> get props => [props];

  @override
  bool? get stringify => true;
}

```
---
### data/datasource
##### To be implemented by all usecases and excepted by repositories
##### `local_datasource.dart`
```
iabstract class LocalTDatasource {
  Future<T> getProperty(String arg);
  Future<void> cacheT(T modelToCache);
}

class LocalTDatasourceImpl implements LocalTDatasource {
  final SharedPreferences sharedPreferences;

  LocalTDatasourceImpl(this.sharedPreferences);

  @override
  Future<T> getProperty(string arg) {
      // override abstract methods in this fashion.
      // local datasource will take advantage of shared prefs
  }
}
```
##### The following example utilizes the `http` package to use a RestAPI as a remote datasource
##### `remote_datasource.dart`
```
abstract class RemoteTDataSource {
  Future<T> getProperty(String arg);
}

class RemoteTDataSourceImpl implements RemoteTDataSource {
  final http.Client client;

  RemoteTDataSourceImpl(this.client);

  @override
  Future<T> getProperty(String arg) async {
    String url = '...';
    http.Response response = await client.get(Uri.parse(url));

    if (response.statusCode == 200) {
      return T.fromJson(jsonDecode(response.body));
    } else {
      throw ServerException();
    }
  }
}
```
---
### data/model
##### Template to be used by any models (replace T with the actual model name)
##### `t_model.dart`
```
T tFromJson(String str) => T.fromJson(json.decode(str));

class T extends Equatable {
  const T({
    required this.prop,
  });

  final A prop;

  static const T empty = T(
    // Whatever empty value corresponds with the datatype
    prop: 0, 
  );

  factory T.fromJson(Map<String, dynamic> json) => T(
        prop: json["prop"],
      );

  Map<String, dynamic> toJson() => {
        "prop": prop,
      };

  @override
  List<Object?> get props => [prop];
}
```
---
### data/repository
##### To implement Repository abstract class from domain layer
##### `t_repository_impl.dart`
```
class RepositoryImpl implements Repository {
    // Include datasources as properties
    // Include any network checking APIs if applicable
}
```
---
### domain/repository
##### To be implemented by Repository classes in data layer
##### `t_repository.dart`
```
abstract class Repository {
  Future<Either<Failure, T>> getProperty(String arg);
}
```
---
### domain/entity
##### Identical to its data-layer model counterpart
##### `t_entity.dart`
```
class TEntity {
  TEntity({
    required this.prop,
  });

  final A prop;

  TEntity copyWith(A? prop) => TEntity(
        prop: prop ?? this.prop,
      );
}
```
---
### domain/usecase
##### To be implemented by Usecase classes
##### `usecase.dart`
```
abstract class UseCase<Type, Params> {
  Future<Either<Failure, Type>> call(Params params);
}
```
---
### presentation/bloc
##### Bloc component files to be exported in `bloc.dart` barrel file.
##### `t_event.dart`
```
abstract class TEvent {}

class GetPropertyEvent extends TEvent {
  GetPropertyEvent(this.prop);

  A prop;
}
```
##### `t_state.dart`
```
enum TStatus { initial, loading, success, failure }

extension TStatusEx on TStatus {
  bool get isInitial => this == TStatus.initial;
  bool get isLoading => this == TStatus.loading;
  bool get isSuccess => this == TStatus.success;
  bool get isFailure => this == TStatus.failure;
}

class TState extends Equatable {
  final TStatus status;
  final T t;

  const TState({
    this.status = TStatus.initial,
    this.t = T.empty,
  });

  TState copyWith(
    TStatus? status,
    T? t,
  ) {
    return TState(
      status: status ?? this.status,
      t: t ?? this.t,
    );
  }

  @override
  List<Object?> get props => [status, t];
}

```
##### `t_bloc.dart`
```
class TBloc extends Bloc<TEvent, TState> {
  final TRepository tRepository;

  TBloc(this.tRepository) : super(const TState()) {
    on<GetPropertyEvent>((event, emit) async {
      GetPropertyUseCase getProperty =
          GetPropertyUseCase(tRepository);
      String arg = event.arg;
      // Set state to LOADING
      emit(const TState(status: TStatus.loading));
      // Fetch t
      Either<Failure, T> result = await getProperty(arg);
      result.fold(
        (left) => emit(
          const TState(status: TStatus.failure),
        ),
        (right) => emit(
          TState(status: TStatus.success, t: right),
        ),
      );
    });
  }
}
```