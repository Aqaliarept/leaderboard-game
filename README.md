# Design decisions

In contrast with a regular stateless web application, the game application is mostly stateful:

- high command rate: it's not possible to store and reload state from the DB for each command from the player
- related data should be placed on same servers for minimize network delays

One of the possible way to deal with such kind stateful applications in respect to arch charecteristics listed below, is an actor model.

## Architectural characteristics

I've defined scalability (elastisity in future), availability and maintainability as the main architecture characteristics.

### Scalability

To cope with high load we have to distribute the load over multiple nodes, and do scale out by increasing nodes count dynamically, based on load.

If the load unevenly distibuted over the time of the day (that is very likely for a game, because of in working hours the load will lower) to save the costs the computation powers should be elastic and shrink when load is decreased.

### Availability

Also during rolled updates and disaters we can loose some nodes and that shouldn't affect user experience.

### Maintainability

The changes and new features should be introduced fast, that is why project must have understandable structure with the short learining curve.

## Decisions

### Commands

For serving the workloads the cluster of actors was chosen ([Proto.Actor](https://proto.actor/) framework) and its [Grains](https://proto.actor/docs/cluster/) . It provides all of the characteristics described above.
The cluster runs a top of Kubernetes, and use it for cluster configuration and discovery.

In additional actor systems provide high throughtput, because active actors are located in memory, and there is no need to get them from database for serving the request.

Actor state persistance (not implemented in the assesment) is achieved by saving events into DB. Because events usuially much smaller than full state that allow update DB much faster, that also affects high throughtput and DB load.

If the node where actor gone offline, the actor is recreated on another available node from the state stored in DB.

### Queries

The proportion queries/commands usually is 5:1. That is why the query side serves querires directly from Redis, without interacting with actor's cluster.

### Maintainability

Project stucture organized according Hexagonal Architecture with folders structured by different layers of application:

- domain
- application
- adapters

# Deploy into kubernetes

The prerequisite for installation is the pre-installed Helm.

- run the command `kubectl get storageclasses` and copy the value of the your cluster storage class name.
- update `stroageClass` value in `leaderboard-helm/values.yaml` with your cluster storage class.
- run `install.sh` in the root of the repo and application will be deployed into `leaderboard` namespace of the current k8s context.

# Testing

Game parameters (competition duration, size, etc) could be confugured by setting up values in `game` section of `leaderboard-helm/values.yaml`

For easier testing competition time is reduced from 1 hour to 200 seconds.

Competition will be started there is at least 2 players in the same level bracket

Level brackets are 0-10, 11-20 and 21-30

There are 30 players with ids from 1 to 30, the player's id corresponds to the player level, e.g. player with id=5 has level=5

Any other players have 1 level.
