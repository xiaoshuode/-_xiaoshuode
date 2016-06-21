# -_xiaoshuode
巧妙地代码机制_xiaoshuode

try catch 机制怎么用，为什么要用，有什么局限性

引用知乎 pig pig的观点！
作者：pig pig
链接：http://www.zhihu.com/question/29459586/answer/44494726
来源：知乎
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

一句话解释：
try catch机制非常好。那些觉得try catch不行的人，是他们自己的水平有问题，无法理解这种机制。并且这群人写代码不遵守规则，喜欢偷懒，这才造成try catch不好的错觉。

详细解释：
1.程序要健壮，必须要设计报错机制。
最古老，也是最常见的，比如：
bool CreateFile( );
//如果创建文件失败就返回false，否则返回true。
这种报错方式，显然不好。因为它没有给出产生错误的具体原因。

2.改进：一个函数或过程，会因为不同的原因产生错误，报错机制必须要把这些错误原因进行区分后，再汇报。
比如：
int CreateFile():
//如果创建成功就返回1.
//如果是因为没有权限，导致失败，返回-1。
//如果是因为文件已经存在，导致失败，返回-2。
//如果是因为创建文件发生超时，导致失败，返回-3。
这样看上去，比【1】要好些，至少指出了比较具体的失败原因，但是，还不够。

3.很多情况下，函数需要把详细的原因，用字符串的方式，返回：
class Result
{
....int State;//同【2】
....string ErrorMessage;//如果失败，这里将给出详细的信息，如果有可能，应该把建议也写上去。
}

Result CreateFile();
//如果创建成功，返回的Result，State为1，ErrorMessage为null。
//如果是因为没有权限，导致失败，返回的Result，State为-1，ErrorMessage为"用户【guest】没有权限在【C:\】这个目录下创建该文件。建议您向管理员申请权限，或者更换具有权限的用户。"。
//如果是因为文件已经存在，导致失败，返回的Result，State为-2，ErrorMessage为"文件【C:\abc.txt】已经存在。如果需要覆盖，请添加参数：arg_overwrite = true"。
//如果是因为创建文件发生超时，导致失败，返回的Result，State为-3，ErrorMessage为"在创建文件时超时，请使用chkdsk检查文件系统是否存在问题。"。

4.我个人推崇上面这种方式，完整，美观。但是这种流程，容易与正常的代码混在一起，不好区分开。因此，Java、C#等设计了try catch这一种特殊的方式：
void CreateFile()
//如果创建成功就不会抛出异常。
//如果是因为没有权限，导致失败，会抛出AccessException，这个Exception的Msg属性为"用户【guest】没有权限在【C:\】这个目录下创建该文件。建议您向管理员申请权限，或者更换具有权限的用户。"。
//如果是因为文件已经存在，导致失败，会抛出FileExistedException，这个Exception的Msg属性为"文件【C:\abc.txt】已经存在。如果需要覆盖，请添加参数：arg_overwrite = true"。
//如果是因为创建文件发生超时，导致失败，会抛出TimeoutException，这个Exception的Msg属性为"在创建文件时超时，请使用chkdsk检查文件系统是否存在问题。"。

可见，上述机制，实际上是用不同的Exception代替了【3】的State。

这种机制，在外层使用时：
try
{
....CreateFile( "C:\abc.txt" );
}
catch( AccessException e )
{
....//代码进入这里说明发生【没有权限错误】
}
catch( FileExistedException e )
{
....//代码进入这里说明发生【文件已经存在错误】
}
catch( TimeoutException e )
{
....//代码进入这里说明发生【超时错误】
}
对比一下【3】，其实这与【3】本质相同，只是写法不同而已。

5.综上，我个人喜欢【3】这类面向过程的写法。但很多喜欢面向对象的朋友，估计更喜欢【4】的写法。然而【3】与【4】都一样。这两种机制都是优秀的错误处理机制。

6.理论说完了，回到正题，题注问：为什么不用try catch？
答：这是因为，很多菜鸟，以及新手，他们是这样写代码的：
void CreateFile( )
//无论遇到什么错误，就抛一个 Exception，并且也不给出Msg信息。
这样的话，在外层只能使用：
try
{
....CreateFile( "C:\abc.txt" );
}
catch( Exception e )
{
....//代码进入这里说明发生错误
}
当出错后，只知道它出错了，并不知道是什么原因导致错误。这同【1】。

以及，即使CreateFile是按【4】的规则设计的，但菜鸟在外层是这样使用的：
try
{
....CreateFile( "C:\abc.txt" );
}
catch( Exception e )
{
....//代码进入这里说明发生错误
....throw Exception( "发生错误" )
}
这种情况下，如果这位菜鸟的同事，调用了这段代码，或者用户看到这个错误信息，也只能知道发生了错误，但并不清楚错误的原因。这与【1】是相同的。

出于这些原因，菜鸟的同事，以及用户，并没有想到，造成这个问题是原因菜鸟的水平太差，写代码图简单省事。他们却以为是try catch机制不行。

因此，这就导致了二逼同事，以及傻比用户，不建议用try catch。

【3】的返回是一个class，如果你需要返回更多信息，直接为这个类增加属性就行。

局限性：
3的确只能处理当层问题。如果用4，4的父级不处理的话，需要在父级方法的注释位置，把所有的Exception都列举一次。以防止外层调用父级，不知道这个问题。


