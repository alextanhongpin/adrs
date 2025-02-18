# Reusable architecture 


After trying many architecture pattern like clean, onion, hexagonal etc, they all doesn't seem to fit the bill.


There are many architecture goals, but they are mostly to satisfy the engineer's preference. There is always these strange tendencies to organize code in a way only one would understand, but that ultimately leads to others getting confused on the execution.

Why is it important then to come up with a standarized code structure?

The reasons are
- lower mental barrier when starting a project
- no digging into the rabbit hole on what's the best practice to do sth
- avoid circling back into other options when one is decided - and justification on why the alternative is unacceptable


What are the goals of the system, and what is considered a good architecture that fulfills it?

- simplicity in reproducing a new service
- template for future projects
- easier testing (aka dependencies are declared as interfaces)
- faster developement
- better monitoring

## CAGE architecture

A modern approach to clean architecture, for golang.

- `controller`: the layer between client and server, executes an action 
- `action`: a verb, aka the business logic
- `gateway`: communicates with external world, such as database, measage queue. ensure atomicity by starting a transaction
- `entity`: the types in our application. holds business logic that can be used globally. returned by gateway

They all belong in the same folder, and are group by feature. This is essentially a form of vertical slicing.

```
authentication
- login.go (action/verb)
- login_controller.go
- register.go (action)
- register_controller.go
- gateway.go
- user.go (entity/noun)
- token.go (entity/noun)
- errors.go
search
- search_controller.go
- search.go
- job.go
apply
- apply.go
- cancel.go
- upload_resume.go
```

## Feature driven

Each directory is a feature.

Each feature contains use cases.

For example, an `authentication` feature may contain the following `actions`:

- login
- register

We refrain from using the word `domain`, as it often leads to confusion. We assume the projects are already created under your domain (e.g. finance/healthcare/education/e-commerce etc).

Most of the system goals above can already be solved through the old way such as interface abstraction that allows mocking in tests etc.


However, we want a more holistic view on the feature adoption, and the usecases.

Take for example, you have `auth` feature, one for password and another `oauth` for social media.

We can compare the performance aka ratio of users that logged in using email/password vs Facebook/Google login.

For each feature we can also track the individual uae case performance, e.g. how many requests/error/duration RED metrics.

We may found that 80% login using whatsapp, and 20% using password.

We may discover that `reset password` usecase frequency is high and consider just implementing passwordless feature.

Then we can compare the adoption, and possible remove reset password if it is not performing.

We also calculate unique counters of logged in user requests per action.
This allows us to measure the adoption rate and compare the month over month grow.

## Shared nothing

Each action is standalone and has its own set of errors.

They are declared in the file together. For common errors, they can be defined in the entity file. But they should be redefined as a separate error in the action file.
