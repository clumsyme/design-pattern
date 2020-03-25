# 创建型模式

- **抽象工厂模式（Abstract Factory）**：提供一个创建一系列相关或相互依赖对象的接口，而无需指定它们具体的类。
- **生成器模式（Builder）**：将一个复杂对象的构建与它的表现分离，使得同样的构建过程可以创建不同的表示。
- **工厂方法（Factory Method）**：定义一个用于创建对象的接口，让子类决定哪一个类实例化。抽象工厂使一个类的实例化延迟到其子类。
- **原型（Prototype）**：用原型实例指定创建对象的种类，并且通过拷贝这个原型来创建新的对象。
- **单例（Singleton）**：保证一个类仅有一个实例，并提供一个访问它的全局访问点。

## 目标示例

考虑一个迷宫创建游戏。迷宫由一系列房间组成，每个房间有其邻居，可能为另一个房间，或者一扇门，或者一堵墙。

每个房间有四面：

```py
class Direction(Enum):
    North=1
    South=2
    East=3
    West=4
```

Room、Wall、Door 继承自 MapSite 类，它有一个方法 `enter`。

```py
from abc import ABC, abstractmethod

class MapSite(ABC):
    @abstractmethod
    def enter(self):
        ...
```

`Room：`

```py
class Room(MapSite):
    def __init__(self, roomNo: int):
        ...

    def getSide(self, direction: Direction) -> MapSite:
        ...

    def setSide(self, direction: Direction, mapSite: MapSite):
        ...

    def enter(self):
        ...
```

`Door`:

```py
class Door(MapSite):
    def __init__(self, room1: Room, room2: Room):
        ...

    def enter(self):
        ...
```

`Wall`:

```py
class Wall(MapSite):
    def __init__(self):
        ...

    def enter(self):
        ...
```

`Maze` 类用来表示房间集合，根据 roomNo 操作可以找到指定房间号的 Room。

```py
class Maze:
    def __init__(self):
        ...

    def addRoom(self, room: Room):
        ...

    def roomNo(self, roomNo) -> Room:
        ...
```

最后是我们的 `MazeGame` 类，我们使用它来创建迷宫。

## `MazeGame.createMaze` 实现

```py
class MazeGame:
    def createMaze(self) -> Maze:
        maze = Maze()
        room1 = Room(1)
        room2 = Room(2)
        door = Door(room1, room2)

        maze.addRoom(room1)
        maze.addRoom(room2)

        room1.setSide(Direction.North, Wall())
        room1.setSide(Direction.East, door)
        room1.setSide(Direction.South, Wall())
        room1.setSide(Direction.West, Wall())

        room2.setSide(Direction.North, Wall())
        room2.setSide(Direction.East, Wall())
        room2.setSide(Direction.South, Wall())
        room2.setSide(Direction.West, door)

        return maze
```

考虑上述代码，它仅仅是创建了两个房间，代码却很复杂。虽然可以将 `setSide` 放到 `Room` 初始化阶段，但这仅仅是将代码移到了其他地方，没有解决这个函数的主要问题：**不灵活**。它对迷宫布局进行了硬编码，改变布局意味着改变这个函数。

创建型模式可以使这个设计更灵活，虽然未必更小。

假设我们想要重用一个迷宫布局，但是这个迷宫是施了魔法的，它有全新的构件如 `DoorNeedingSpell`，需要咒语才能打开。`EnchantedRoom`: 可以有魔法钥匙或者咒语的房间。我们怎么才能复用 `createMaze` 的迷宫布局呢。

加入我们不使用创建型模式，那么我们需要改变硬编码：

```diff
- room1 = Room(1)
+ room1 = EnchantedRoom(1)
```

使用创建型模式，我们可以去除实例化这些构件对这些类的具体引用来增加灵活性：

1. 如果 MazeGame 调用方法而不是构造器来创建他所需要的房间、墙、门，那么我们可以创建 MazeGame 的子类来重定义这些方法来改变被实例化的类。

```diff
class MazeGame:
+   def makeRoom(roomNo: int):
+       return Room(roomNo)
    def createMaze(self):
        ...
-       room1 = Room(1)
+       room1 = self.makeRoom(1)
        ...

+ class MagicMazeGame(MazeGame):
+     def makeRoom(roomNo: int):
+         return EnchantedRoom(roomNo)

maze = MazeGame().createMaze()
magicMaze = MagicMazeGame().createMaze()
```

这就是`工厂方法模式`。

2. 如果传递一个对象给 `createMaze` 方法，由这个对象来创建房间、墙、门，那么可以通过传递不同的对象来实现创建不同的房间等。

```diff
class MazeGame:
    def createMaze(self, factory) -> Maze:
        ...
-       room1 = Room(1)
+       room1 = factory.makeRoom(1)

+ class MazeFactory:
+     ...
+     def makeRoom(self, roomNo: int) -> Room:
+         return Room(roomNo)
+ 
+ class MagicMazeFactory(MazeFactory):
+     ...
+     def makeRoom(self, roomNo: int) -> Room:
+         return EnchantedRoom(roomNo)

maze = MazeGame.createMaze(MazeFactory())
magicMaze = MazeGame.createMaze(MagicMazeFactory())
```

这就是 `虚拟工厂模式`。