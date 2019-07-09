2. pt-fifo-split

功能介绍：

模拟切割文件并通过管道传递给先入先出队列而不用真正的切割文件

用法介绍：

pt-fifo-split [options] [FILE ...]
pt-fifo-split 读取大文件中的数据并打印到 fifo 文件， 每次达到指定行数就往fifo 文件中打印一个 EOF 字符，读取完成以后，关闭掉fifo 文件并移走，然后重建 fifo 文件，打印更多的行。 这样可以保证你每次读取的时候都能读取到制定的行数直到读取完成。注意此工具只能工作在类unix 操作系统。 这个程序对大文件的数据导入数据库非常有用。

使用示例：

范例 1：一个每次读取一百万行记录的范例：

pt-fifo-split --lines 1000000 hugefile.txt
while [ -e /tmp/pt-fifo-split ]; do cat /tmp/pt-fifo-split; done
范例 2： 一个每次读取一百万行，指定 fifo 文件为/tmp/my-fifo，并使
用load data 命令导入到 mysql 中：
pt-fifo-split infile.txt --fifo /tmp/my-fifo --lines 1000000
while [ -e /tmp/my-fifo ]; do
mysql -e "set foreign_key_checks=0; set sql_log_bin=0; set uniq
ue_checks=0; load data local infile '/tmp/my-fifo' into table load_t
est fields terminated by '\t' lines terminated by '\n' (col1, col2);"
sleep 1;
done
