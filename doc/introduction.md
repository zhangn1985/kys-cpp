## 一些代码的说明

### 公共部分

Engine封装了一套SDL2的主要实现，与SmallPot类似。如更换绘图引擎，则只需修改此部分即可。

Save中对所有数据进行了封装，可以较为方便地调用。

TextureManger是一个纹理管理器，因为《金庸群侠传》的贴图是含偏移设置的，故有些特殊的地方。

Audio是音频类，基于BASS或者SDL_mixer，可以播放mid、mp3、wav等格式。

PotConv封装了iconv的实现。

### RunNode

RunNode是游戏中的基本执行类，包含5个重要的虚函数：backRun，draw，dealEvent，onEnter，onExit。对应在背景中执行，如何画自身，如何处理事件，进入时的处理，退出时的处理。一般来说，衍生类应重写这些函数。

其中每个节点可以包含数个子节点，在绘图时子节点也会被自动一一绘出。注意在画自身的部分不需要处理子节点，除非有特殊的需要。

存在一个全局的的RunNode栈root（实际是std::vector），会从下到上依次画出每个节点。RunNode类有一个占满全屏的属性，表示这个类将占用全部的屏幕，因此引擎在绘制的时候，会仅找出最靠上的含有该属性的节点，并从这里开始往上画。

创建一个节点，并调用run过程即可运行此节点，注意使用run执行的节点是完全独占的，其子节点也会有事件响应。如果需要退出当前节点，在适当的地方使用setExit(true)即可，但是子节点调用是无效的，除非拥有当前运行节点的指针。

run过程的参数为一个布尔值，如果为true则会被加入到root并进行绘制，如果为false则只运行不参与绘制。但是很多节点的draw过程是空的，即使放到root中也不会参与绘制，实际利用了这一特性的仅有显示人物对话的部分。

run过程会返回一个函数值，可以利用进行一些判断，例如菜单的选择。

规定所有节点均使用共享指针，可以比较自由地互相包含。请不要让子节点出现递归包含，这样会迅速消耗掉所有资源。

通常来说，大部分游戏引擎都需要全局标记和回调来控制剧情的执行，本框架的设计在绘图无阻塞执行的同时，事件仍是以阻塞的模式顺序执行的，这样无需额外的事件标记。

## 资源的保存以及abc工程

存档的基础数据部分（即r部分）可以保存为sqlite的数据库格式。可以通过读取和保存来转换已有存档。

游戏的资源文件是以单个图片的形式放在resource的各个目录中的，每张图的偏移保存在index.ka中，格式为每张图两个16位整数，连续存放。这种类型的目录也可以打包为zip格式。

战斗贴图文件中，每个人的帧数，之前在hugebase（水浒）框架中使用fightframe.ka保存，现改用fightframe.txt保存。格式为动作索引（0~4），每方向数量。未写则视为0。这种类型的目录也可以打包为zip格式。

之前游戏使用的列表文件只保留了升级经验列表和离队列表，改用txt格式。

存档文件的文本编码，仅有初始存档为cp950（BIG5），这是向下兼容的需要，但是内部会使用65001（utf-8），存档被保存后也会转为65001。存档中有一个标记位保存了该文件的编码。

abc工程用来转换之前的数据。建议自行调整代码后，使用调试模式执行。

其中主要的功能是将存档的R部分扩展为原来的二倍。即所有的16位整数转为32位整数，表示范围从32767扩大到2^31-1，足够通常的数值使用。同时，原有的字串也扩展为之前的二倍长度，例如原来人物的名字有5个中文字符长度，实际上最多只能使用4个字，转换之后因字符串的编码使用了utf-8，可以使用6个汉字（并不是推荐你用6个字）。但是如果使用数据库来存档，则长度不受此限制。转换之后的文件名变为r?.grp32。

并非所有的文档都转为32位，部分是为了节省资源的需要，以及某些情况过于复杂。

以上提到的数据，除了文本文件外均可以用真正的强强的新版upedit修改（该修改器不完善）。