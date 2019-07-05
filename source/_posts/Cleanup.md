---
title:  cleanup 自动资源管理
---
没有麻烦和安全地调用您的close()方法.

#### Overview
可以使用@Cleanup以确保在代码执行路径退出当前作用域之前自动清除给定资源。您可以通过使用@Cleanup注释任何局部变量声明来执行此操作：
@Cleanup InputStream in = new FileInputStream("some/file");
结果，在您作用域范围的末尾调用in.close()。保证通过try / finally构造运行此调用。请看下面的示例，看看它是如何工作的。
如果要清理的对象类型没有close()方法，但是有其他一些无参数方法，则可以指定此方法的名称，如下所示：
@Cleanup("dispose") org.eclipse.swt.widgets.CoolBar bar = new CoolBar(parent, 0);
默认情况下，清除方法被假定为close()。@Cleanup无法调用带有1个或多个参数的清理方法。

#### With Lombok

    import lombok.Cleanup;
    import java.io.*;
    
    public class CleanupExample {
      public static void main(String[] args) throws IOException {
        @Cleanup InputStream in = new FileInputStream(args[0]);
        @Cleanup OutputStream out = new FileOutputStream(args[1]);
        byte[] b = new byte[10000];
        while (true) {
          int r = in.read(b);
          if (r == -1) break;
          out.write(b, 0, r);
        }
      }
    }
    
#### Vanilla Java

    import java.io.*;
    
    public class CleanupExample {
      public static void main(String[] args) throws IOException {
        InputStream in = new FileInputStream(args[0]);
        try {
          OutputStream out = new FileOutputStream(args[1]);
          try {
            byte[] b = new byte[10000];
            while (true) {
              int r = in.read(b);
              if (r == -1) break;
              out.write(b, 0, r);
            }
          } finally {
            if (out != null) {
              out.close();
            }
          }
        } finally {
          if (in != null) {
            in.close();
          }
        }
      }
    }
    
#### Supported configuration keys:
lombok.cleanup.flagUsage = [warning | error] (default: not set)
如果配置， Lombok会将任何@Cleanup用法标记为警告或错误。
#### Small print
在finally块中，只有在给定资源不是null的情况下才会调用cleanup方法。但是，如果您使用delombok在代码上，则插入lombok.Lombok.preventNullAnalysis(Object o)调用以防止警告，如果静态代码分析可以确定，则不需要进行 null-check。使用 lombok.jar 类路径进行编译会删除该方法调用，因此不存在运行时依赖性。
如果您的代码抛出异常，并且随后触发的清理方法调用也会抛出异常，则清理调用抛出的异常将隐藏原始异常。你不应该依赖这个“功能”。最好是，lombok想生成代码，这样，如果主体抛出了异常，那么关闭调用抛出的任何异常都会被静默吞噬（但如果主体以任何其他方式退出，则关闭调用的异常将不会是吞咽）。lombok的作者目前不知道实现这个方案的可行方法，但是如果java更新允许它，或者我们找到了一种方法，我们将修复它。
