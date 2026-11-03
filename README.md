# Claw is a statically typed systems level language focused around readability and ease of learning.

I wanted a language that could compare in performance to C, without the difficulty and risks of such a language. This is mainly because I like having small and fast binaries for ui applications, but I don't like the boilerplate and macro-heavy solutions in the current low level economy. Claw is meant to be a very versatile language, supporting frontend applications as well as libraries for the backend.

The idea here is that we let the compiler do the vast majority of the work. This idea is not new obviously, but Claw takes some unique approaches to maintain the simplicity of a highly abstracted language.

A lot of claw follows in Zig's footsteps, we inherit the principles of avoiding hidden allocations. And types are a compile time value.
But we bring duck typing to a new level with our interface system, and events make control easier and more powerful without overhead or metaprogramming.

Claw's main stake in the language ecosystem is implicit interfaces and events.

## Any struct can implicitly conform to other structs if it has at least the methods and fields of the interface struct.

```claw
import std;

pub const Parent = struct {
  field: int,
  otherField: int,

  fn |self: Parent*| method() void {

  }
};

// alias makes it inherit fields and methods from the Parent 
// struct
pub const Child = alias Parent {
  # capture parameter can be Child since it conforms to Parent 
  # struct.
  override fn |self: Child*| method() void {

  }
};

pub fn printParent(param: Parent) void {
  // uses similar syntax to zig here since param count must be known 
  // at comptime
  std.io.printf("field: %d, other: %d\n", .{ param.field, param.otherField });
}

pub fn main() void {
  const child: Child = .{
    .field = 0,
    .otherField = 10,
  };

  // this works because the compiler can prove that Child has 
  // neccessary fields and methods of the interface requested.
  // 
  // internally a function is generated for each struct that gets 
  // input here. We dont really care about the type, we just care 
  // that it conforms
  printParent(child);
}

// Note:
//
// alias does not need to be used. It is just an easy way to ensure
// that a type has all required fields and methods.
// you could use reflection to do the same, or manually handle it 
// if you know what you are trying to conform to.
```

## Events are just as powerful. And feel unreal to work with in such a low level context.
```claw
import std;
import build; //compiler generates the build fields from build.clon

pub const App = struct {
  name = build.name,
  version = build.version,
  args: std.String[],
  window: std.ui.Window*,

  allocator: std.mem.Allocator,

  # events dont take any parameters or return anything (except errors)
  #
  # you define an event and what it triggers, the compiler resolves what to call
  # and when to call it.
  # events can be overridden by other structs that alias this struct
  event |self: App*| start {
    self.window.title = self.name; //set title name
    app.window.show(); //show window
  }
};

pub fn main(args: std.String[]) void {
  const allocator = std.Allocators.DebugAllocator;

  app: App = .{
    .args = args,
    .allocator = allocator,

    // using events as the groundwork an optional garbage collector
    // can be defined as part of the standard using reference counting
    // a threadsafe version is available through std.gc.createAtomic(T: Destroyable, allocator: Allocator)
    // Destroyable interface just requires a deinit function (make an empty one if not needed) to be called
    // when the memory is decided unused.
    .window = try std.gc.create(std.ui.Window, allocator),
  };

  // this will be called after the event body is called
  // from in the event definition.
  on app.start {
    std.io.println("Window Shown!");
  }

  do app.start; //call start event
}
```
