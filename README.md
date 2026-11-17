# Risk

A Java console implementation of classic World Domination Risk for 3-6 players.
Includes territory claiming, reinforcements, card trading, attacks, territory capture,
player elimination, fortification, and victory when one player controls all 42 territories.

## Contributors
- Anant Poddar
- Vihaan Shah
- Aanand Patel

## Dependencies
- JDK 11
- JUnit 5.10
- Gradle 8.10

## Running the Game

```sh
./gradlew classes
java -cp build/classes/java/main:build/resources/main code.Main
```

## Testing

```sh
./gradlew test -x pitest
./gradlew checkstyleMain checkstyleTest
```

Boundary value analyses are in `docs/bva/`, and design documentation is in `docs/design/`.
