Integrated Itinerary and Fare Engine – Medellín Metro System
Practice 2 – Programming Languages and Paradigms (SI1002), Universidad EAFIT.
Team members
Juan José Bedoya Zapata
Andrés Stiven Copete Montoya
How to build and run
Requires g++ with C++17 support.
```bash
g++ -std=c++17 -Wall -Wextra -o practice main.cpp
./practice
```
The program compiles with zero errors and zero warnings. It also runs on
OnlineGDB with the language set to C++17.
Defence questions
Question 1 – Why must the destructor of `TransportMode` be virtual? What happens without it?
The network stores its modes as `std::unique_ptr<TransportMode>`, so every
object is destroyed through a pointer to the base class. If `~TransportMode()`
is not virtual, `delete` on a base pointer that points to a derived object is
undefined behaviour according to the C++ standard. In practice the destructor
of the derived class (for example `TouristCable`) never runs: only the base
part is destroyed, so any resource owned by the derived part is leaked and its
members are never cleaned up. Declaring `virtual ~TransportMode() = default;`
makes the destruction go through the vtable, so the most derived destructor
runs first and then the base destructors in reverse order of construction.
It is a correctness requirement, not a formality.
Question 2 – Why an abstract class with pure virtual functions instead of an enumeration plus a `switch`?
With an enum and a `switch`, the knowledge of "how each mode behaves" is spread
over every function that switches on the type (fare, student policy,
integration, description). Adding a new mode, such as a new kind of cable,
forces us to open and modify all of them, and the compiler does not warn us if
we forget one. With an abstract `TransportMode` and the pure virtual functions
`baseFare()` and `serviceType()`, plus the hooks `allowsStudentFlatFare()`,
`allowsIntegration()` and `opensIntegrationWindow()`, each behaviour lives in
the class that owns it. A new mode is a new class: `StudentPassenger`,
`chargeItinerary` and the reports are not touched (open/closed principle).
`TouristCable` shows it: it overrides three hooks and reuses `CableLine::baseFare()`
instead of anyone checking "is this a tourist cable?" (Rule A: no behaviour is
decided by the type of an object). Pure virtual functions also make the class
abstract, so the compiler stops anyone from creating a mode that has no fare.
