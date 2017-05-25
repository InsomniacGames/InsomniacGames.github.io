Pretty Little State Machine
===========================

I recently stumbled upon a way to implement a state machine that is much simpler
than anything I have used before. I haven’t seen it done this way, so I’d like
to share it.

The problem I tried to solve involved a user interface to remove a portion of
one asset and save it as a stand-alone asset. Our tools don’t write to files on
disk but use a database. The desired sequence required several round trips to
the asset database to see if the assets exist, create them if not, check if they
are writable, prompt the user to check them out from the Perforce depot if not,
and so on. Details are not important, only the fact that there were multiple
stages that involved round trips to the server, user prompts and dialogs, and
that all of this had to be done asynchronously and event based.

I could not write a linear function and lock up the thread, so a state machine
was called for.

I know state machine libraries exist but in most cases, including this one, I
find they overcomplicate the issue. An ad-hoc state machine will work just fine.
I would usually write a state machine like this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
class State
{
public:
  virtual void enter() {}
  virtual void exit() {}
};

class StateFoo final : public State
{
public:
  virtual void enter() override {/* set up foo */}
  virtual void exit() override {/* clean up foo */}
};

State    myStateNull;
StateFoo myStateFoo;

State* currentState = &myStateNull;
void SetState(State* s)
{
  currentState->exit();
  currentState = s;
  currentState->enter();
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

...and so on. I will assume that you are familiar with this approach.

There are several aspects of this approach that I don’t like. It requires a
significant amount of boilerplate code per state: a class definition with all
its trappings and the instantiation of an object. Sharing and passing data
between states, in my case the source and destination asset names, creates
further complexity. It’s perfectly doable, quite reasonable, done it for years,
but I found a better way.

In the new approach, instead of one class per state, I only need a function per
state. No base class, no inheritance, no state objects. It looks like this:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
enum class Operation { kEnter, kExit };
using StateFn = void(Operation);

void StateNull(Operation op) {}

void StateFoo(Operation op)
{
  if (op == Operation::kEnter) {/* set up foo */}
  else                         {/* clean up foo */}
}

StateFn* currentState = &StateNull;
void SetState(StateFn* s)
{
  currentState(Operation::kExit);
  currentState = s;
  currentState(Operation::kEnter);
}
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the examples above, states are declared at the global scope. In my final
implementation, states are member functions. The only class involved is the
state machine itself. When the operation I described (remove and save a portion
of an asset) is triggered, the state machine is created. On entering, each state
posts requests or dialogs and hooks up response or event handlers as needed. On
exit, the state will unhook the handlers. When the state machine is done, it
deletes itself.

Because states are just member functions, they have access to all data members
of the state machine.

These are the first few lines of my final state machine, slightly abridged:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
class PrefabCreatorStateMachine : public QObject
{
  Q_OBJECT
  enum class Operation { kEnter, kExit };
  using StateFn = void (PrefabCreatorStateMachine::*)(Operation op);
  void stateNull(Operation) {}
  m_StateFn = &PrefabCreatorStateMachine::stateNull;
  void setState(StateFn state_fn)
  {
    (this->*m_StateFn) (Operation::kExit);
    m_StateFn = state_fn;
    (this->*m_StateFn) (Operation::kEnter);
  }
  void finish()
  {
    setState(&PrefabCreatorStateMachine::stateNull);
    delete this;
  }
// etcetera, etcetera
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Our tools are made with Qt, but this state machine pattern does not require it.
Speaking to fellow Qt users for a moment: the “enter” operation of a state
typically connects state specific slots, and posts requests. The slot
implementation changes the state. The “exit” operation of a state disconnects
everything that the “enter” connected. Both “enter” and “exit” operations may
emit signals.

I have found this way of implementing a state machine the most straightforward,
the most compact and most readable I have ever used.
