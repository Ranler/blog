title: Shell Style
date: 2013-12-24 08:57:54
categories: Shell
---

From Google styleguide: [Shell Style Guide](http://google-styleguide.googlecode.com/svn/trunk/shell.xml)

### 1. 环境

#### 1.1 使用Shell类型

只使用`Bash`，文件以`#!/bin/bash`开头。
使用`set`来设置shell以便`bash <script_name>`正常执行。

#### 1.2 使用Shell环境

- Shell适合作为胶水语言去调用其它程序而不是进行数据处理；
- Shell适合对程序性能要求不高的情况；
- Shell适合处理简单的变量如`${PIPESTATUS}`，如果要使用数组，请使用Python；
- Shell适合写不超过100行的脚本，否则应该使用Python。


### 2. 文件和解释器调用

#### 2.1 文件扩展名

**可执行文件**推荐不写文件扩展名，也可以以`.sh`结尾。
最终用户不需要知道可执行文件是什么语言写的。

**库文件**必须以`.sh`结尾并且没有可执行权限。
因为库文件面向的是开发者，不同语言的开发者需要使用相应语言的库文件。

#### 2.2 禁止SUID和SGID

SUID和SGID会给用户以文件拥有者和组的权限执行。
这会造成很多安全问题，所以禁止使用`SUID`和`SGID`。

当需要给予用户更高的权限时，使用`sudo`的方式。


### 3. 环境

#### 3.1 STDOUT vs STDERR

所有的错误信息必须定向到`STDERR`。
这样方便从正常日志中分离出异常信息。

推荐使用函数输出异常信息：

``` shell
err() {
  echo "[$(date +'%Y-%m-%dT%H:%M:%S%z')]: $@" >&2
}
  
if ! do_something; then
  err "Unable to do_something"
  exit "${E_DID_NOTHING}"
fi
}
```

### 4. 注释

#### 4.1 文件开头

内容注释，也可以加上Copyright和作者信息：

``` shell
#!/bin/bahs
#
# Perform hot backups of Oracle databases.
```

#### 4.2 函数注释

无论函数的长短或复杂度怎样，都必须添加注释。
完整的注释包括：

- 函数描述
- 全局变量的使用和修改
- 参数
- 返回值而不是函数最后一行的默认程序退出状态

例如：

``` shell
#!/bin/bash
#
# Perform hot backups of Oracle databases.

export PATH='/usr/xpg4/bin:/usr/bin:/opt/csw/bin:/opt/goog/bin'

#######################################
# Cleanup files from the backup dir
# Globals:
#   BACKUP_DIR
#   ORACLE_SID
# Arguments:
#   None
# Returns:
#   None
#######################################
cleanup() {
  ...
  }
}
```

#### 4.3 实现注释

注释脚本中重要的、复杂的、不清楚的部分。

#### 4.4 TODO注释

例如：

``` shell
# TODO(mrmonkey): Handle the unlikely edge cases (bug ####)
```

### 5. 格式

#### 5.1 缩进

缩进2个空格，不要使用tab缩进。

#### 5.2 行长度和长字符串

每行不超过80个字符。

长字符串使用下面方式：

``` shell
# DO use 'here document's
cat <<END;
I am an exceptionally long
string.
END

# Embedded newlines are ok too
long_string="I am an exceptionally
  long string."
```

### 5.3 管道

如果只有一个管道，那么放在同一行，否则每行放置一个。
`||`和`&&`同样适用此规则。

``` shell
# All fits on one line
command1 | command2

# Long commands
command1 \
  | command2 \
  | command3 \
  | command4
```

### 5.4 循环

把`; do`和`; then`放在和`while`,`for`,`if`同一行。
`else`和`done`单独一行。

``` shell
for dir in ${dirs_to_cleanup}; do
  if [[ -d "${dir}/${ORACLE_SID}" ]]; then
    log_date "Cleaning up old files in ${dir}/${ORACLE_SID}"
    rm "${dir}/${ORACLE_SID}/"*
    if [[ "$?" -ne 0 ]]; then
      error_message
    fi
  else
    mkdir -p "${dir}/${ORACLE_SID}"
    if [[ "$?" -ne 0 ]]; then
      error_message
    fi
  fi
done
```

### 5.5 case语句

- 只有一行分支语句的话，`;;`可以紧接同一行；
- 有多行分支语句的话，`;;`必须单独一行；

``` shell
case "${expression}" in
  a)
    variable="..."
    some_command "${variable}" "${other_expr}" ...
    ;;
  absolute)
    actions="relative"
    another_command "${actions}" "${other_expr}" ...
    ;;
  *)
    error "Unexpected expression '${expression}'"
    ;;
esac
```

``` shell
verbose='false'
aflag=''
bflag=''
files=''
while getopts 'abf:v' flag; do
  case "${flag}" in
    a) aflag='true' ;;
    b) bflag='true' ;;
    f) files="${OPTARG}" ;;
    v) verbose='true' ;;
    *) error "Unexpected option ${flag}" ;;
  esac
done
```

### 5.6 变量

- 变量尽量加上大括号
- 特殊变量无需加大括号：`$1`,`$@`,`$*`...

``` shell
# Section of recommended cases.

# Preferred style for 'special' variables:
echo "Positional: $1" "$5" "$3"
echo "Specials: !=$!, -=$-, _=$_. ?=$?, #=$# *=$* @=$@ \$=$$ ..."

# Braces necessary:
echo "many parameters: ${10}"

# Braces avoiding confusion:
# Output is "a0b0c0"
set -- a b c
echo "${1}0${2}0${3}0"

# Preferred style for other variables:
echo "PATH=${PATH}, PWD=${PWD}, mine=${some_var}"
while read f; do
  echo "file=${f}"
done < <(ls -l /tmp)
  
# Section of discouraged cases
  
# Unquoted vars, unbraced vars, brace-quoted single letter
# shell specials.
echo a=$avar "b=$bvar" "PID=${$}" "${1}"
  
# Confusing use: this is expanded as "${1}0${2}0${3}0",
# not "${10}${20}${30}
set -- a b c
echo "$10$20$30"
```

### 5.7 引用

- 引用的对象包含变量、子命令、空格、Shell关键字(`$`)
- 多个词组成的变量名需要引用
- 整数不需要引用
- 尽量使用`$@`而不是`$@`

``` shell
# 'Single' quotes indicate that no substitution is desired.
# "Double" quotes indicate that substitution is required/tolerated.

# Simple examples
# "quote command substitutions"
flag="$(some_command and its args "$@" 'quoted separately')"

# "quote variables"
echo "${flag}"

# "never quote literal integers"
value=32
# "quote command substitutions", even when you expect integers
number="$(generate_number)"

# "prefer quoting words", not compulsory
readonly USE_INTEGER='true'

# "quote shell meta characters"
echo 'Hello stranger, and well met. Earn lots of $$$'
echo "Process $$: Done making \$\$\$."

# "command options or path names"
# ($1 is assumed to contain a value here)
grep -li Hugo /dev/null "$1"

# Less simple examples
# "quote variables, unless proven false": ccs might be empty
git send-email --to "${reviewers}" ${ccs:+"--cc" "${ccs}"}

# Positional parameter precautions: $1 might be unset
# Single quotes leave regex as-is.
grep -cP '([Ss]pecial|\|?characters*)$' ${1:+"$1"}

# For passing on arguments,
# "$@" is right almost everytime, and
# $* is wrong almost everytime:
#
# * $* and $@ will split on spaces, clobbering up arguments
#   that contain spaces and dropping empty strings;
# * "$@" will retain arguments as-is, so no args
#   provided will result in no args being passed on;
#   This is in most cases what you want to use for passing
#   on arguments.
# * "$*" expands to one argument, with all args joined
#   by (usually) spaces,
#   so no args provided will result in one empty string
#   being passed on.
# (Consult 'man bash' for the nit-grits ;-)

set -- 1 "2 two" "3 three tres"; echo $# ; set -- "$*"; echo "$#, $@")
set -- 1 "2 two" "3 three tres"; echo $# ; set -- "$@"; echo "$#, $@")
```

### 6. 特点和Bugs

#### 6.1 子命令

使用`$(command)`代替倒引号\`command\`，前者在嵌套时更容易读。

``` shell
# This is preferred:
var="$(command "$(command1)")"

# This is not:
var="`command \`command1\``"
```

#### 6.2 测试语句

使用`[[ ... ]]`代替`[`,`test`,`/usr/bin/[`。
`[[ ... ]]`允许使用正则表达式。

``` shell
# This ensures the string on the left is made up of characters in the
# alnum character class followed by the string name.
# Note that the RHS should not be quoted here.
# For the gory details, see
# E14 at http://tiswww.case.edu/php/chet/bash/FAQ
if [[ "filename" =~ ^[[:alnum:]]+name ]]; then
  echo "Match"
fi
  
# This matches the exact pattern "f*" (Does not match in this case)
if [[ "filename" == "f*" ]]; then
  echo "Match"
fi
	
# This gives a "too many arguments" error as f* is expanded to the
# contents of the current directory
if [ "filename" == f* ]; then
  echo "Match"
fi
```

### 6.3 测试字符串

使用`[[ ... ]]`的`-z`,`-n`参数来测试字符串，而不是与空字符串比较。

``` shell
# Do this:
if [[ "${my_var}" = "some_string" ]]; then
  do_something
fi
  
# -z (string length is zero) and -n (string length is not zero) are
# preferred over testing for an empty string
if [[ -z "${my_var}" ]]; then
  do_something
fi
	
# This is OK (ensure quotes on the empty side), but not preferred:
if [[ "${my_var}" = "" ]]; then
  do_something
fi
	  
# Not this:
if [[ "${my_var}X" = "some_stringX" ]]; then
  do_something
fi

# Use this
if [[ -n "${my_var}" ]]; then
  do_something
fi
		  
# Instead of this as errors can occur if ${my_var} expands to a test
# flag
if [[ "${my_var}" ]]; then
  do_something
fi
```

### 6.4 文件名的通配符扩展

因为文件名可以以`-`开头，因此使用`./*`代替`*`更安全。

``` shell
# Here's the contents of the directory:
# -f  -r  somedir  somefile

# This deletes almost everything in the directory by force
psa@bilby$ rm -v *
removed directory: `somedir'
removed `somefile'

# As opposed to:
psa@bilby$ rm -v ./*
removed `./-f'
removed `./-r'
rm: cannot remove `./somedir': Is a directory
removed `./somefile'
```			

### 6.5 避免使用`eval`

`eval`使得其后面的返回结果难以检查。

``` shell
# What does this set?
# Did it succeed? In part or whole?
eval $(set_my_variables)

# What happens if one of the returned values has a space in it?
variable="$(eval some_function)"
```

### 6.6 Pipes to While

使用子程序或for循环代替pipes to while.
因为循环执行在子shell内，子shell中的变量修改不会影响到当前shell变量。
例如：

``` shell
last_line='NULL'
# 下面这个pipes to while是在子shell中执行的，
# 因此last_time的改变不影响当前shell的last_time变量
your_command | while read line; do
  last_line="${line}"
done
  
# This will output 'NULL'
echo "${last_line}"
```

如果command结果不包含空格或特使字符，可使用for循环实现：

``` shell
total=0
# Only do this if there are no spaces in return values.
for value in $(command); do
  total+="${value}"
done
```

也可以显式使用子shell执行command，并把结果重定向到while中：

``` shell
total=0
last_file=
while read count filename; do
  total+="${count}"
    last_file="${filename}"
done < <(your_command | uniq -c)
	
# This will output the second field of the last line of output from
# the command.
echo "Total = ${total}"
echo "Last one = ${last_file}"
```

### 7. 命名规则

#### 7.1 函数名

小写字母，下划线连接。
使用`::`连接库名和函数名。
`function`关键字可有可无，但必须统一。

``` shell
# Single function
my_func() {
  ...
}
  
# Part of a package
mypackage::my_func() {
  ...
}
```

#### 7.2 变量名

同函数名。

#### 7.3 常量和环境变量名

字母大写、下划线连接、在文件首部声明。

``` shell
# Constant
readonly PATH_TO_FILES='/some/path'

# Both constant and environment
declare -xr ORACLE_SID='PROD'
```

有些变量在开始时并不是只读（如`getops`处理的变量），
因此可以在变量赋值完成后把他们设为只读。

``` shell
VERBOSE='false'
while getopts 'v' flag; do
  case "${flag}" in
    v) VERBOSE='true' ;;
  esac
done
readonly VERBOSE
```

当在函数中设置全局变量时，可以用`readonly`或`export`代替`declare`。


#### 7.4 源文件名

字母小写、下划线连接。

#### 7.5 只读变量

使用`readonly`或`declare -r`声明变量为只读。

``` shell
zip_version="$(dpkg --status zip | grep Version: | cut -d ' ' -f 2)"
if [[ -z "${zip_version}" ]]; then
  error_message
else
  readonly zip_version
fi
```

#### 7.6 函数局部变量

使用`local`声明局部变量。
局部变量的声明和赋值应该分成两行，因为不能记录赋值子命令的返回状态码。

``` shell
my_func2() {
  local name="$1"
  
  # Separate lines for declaration and assignment:
  local my_var
  my_var="$(my_func)" || return
		
  # DO NOT do this: $? contains the exit code of 'local', not my_func
  local my_var="$(my_func)"
  [[ $? -eq 0 ]] || return

  ...
}
```

#### 7.7 本地函数

把所有函数定义集中放在常量后面，禁止把执行语句和函数定义混合排列。
在函数前只可以包含includes，`set`语句，常量定义。

#### 7.8 main

对于很长的脚本，最好写main函数，并在源文件的最后一行调用：

``` shell
main "%@"
```

### 8. 调用命令

#### 8.1 检查返回值

一定要检查返回值。

对于非管道命令，使用`$?`或`if`来检查：

``` shell
if ! mv "${file_list}" "${dest_dir}/" ; then
  echo "Unable to move ${file_list} to ${dest_dir}" >&2
exit "${E_BAD_MOVE}"
fi
	
# Or
mv "${file_list}" "${dest_dir}/"
if [[ "$?" -ne 0 ]]; then
  echo "Unable to move ${file_list} to ${dest_dir}" >&2
  exit "${E_BAD_MOVE}"
fi
```

对于管道命令，Bash使用`PIPESTATUS`变量记录返回值。
比如直接检查整个管道命令是否成功：

``` shell
tar -cf - ./* | ( cd "${dir}" && tar -xf - )
if [[ "${PIPESTATUS[0]}" -ne 0 || "${PIPESTATUS[1]}" -ne 0 ]]; then
  echo "Unable to tar files to ${dir}" >&2
fi
```

如果要对管道中每个命令的进行不同处理，则要首先保存`PIPESTATUS`值，
以防被覆盖（`[`命令就会覆盖`PIPESTATUS`）：

``` shell
tar -cf - ./* | ( cd "${DIR}" && tar -xf - )
return_codes=(${PIPESTATUS[*]})
if [[ "${return_codes[0]}" -ne 0 ]]; then
  do_something
fi
if [[ "${return_codes[1]}" -ne 0 ]]; then
  do_something_else
fi
```

#### 8.2 内建命令vs.外部命令

首选Shell内建命令，比如`bash(1)`中的Parameter Expansion functions。

``` shell
# Prefer this:
addition=$((${X} + ${Y}))
substitution="${string/#foo/bar}"

# Instead of this:
addition="$(expr ${X} + ${Y})"
substitution="$(echo "${string}" | sed -e 's/^foo/bar/')"
```



