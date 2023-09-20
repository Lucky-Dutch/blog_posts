## 5 LiveView tips and tricks.
### First Tip
#### Use functional components (stateless) wherever you can, use live_components (stateful) when a component needs a state modification.
Why?
Stateful components create a child live view and if we do not need to manage the state, we should try to avoid using them. It's like using a gun to kill a fly.
### Second Tip
#### Instead of assigning new states, it is better to update them.
if you are sure that the state exists in the liveview, i.e. you added the state in mount/2 and then want to change it, e.g. in handle_event/2, instead of re-assigning it, use the update function
