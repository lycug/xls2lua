# xls2lua

将excel文件(.xls, .xlsx)转换为lua代码的小工具.

## 使用方法

### 1. 独立运行:

```sh
./xls2lua.py *.xlsx
```

### 2. 作为库导入

这种方式可以实现一些自定义功能,比如在生成的lua中额外插入代码:

```py
import xls2lua

def _my_writer(sheet_name, lua_path, code):
    code += u'''
-- insert some code --
for key, node in pairs(sheet) do
    print(key);
end
'''
    open(lua_path, "wb").write(code.encode("utf-8"));

converter = xls2lua.Converter();
converter.convert("test1.xlsx", _my_writer);
```

## 主要特性:
- 支持多级索引转换,生成层级table
- 也支持无索引转换,按行生成数组
- 支持差量转换,只转换有变更的文件(基于文件哈希比较)
- 支持把excel列标题直接做lua变量名,也支持用额外metadata指定变量名和变量类型(比如列标题可能是中文的)
- 支持指定转换的数据类型,如string,number,bool,也可以不指定类型
- 支持在生成的lua代码前后额外插入代码(比如用来在加载时做预处理)

## 环境需求

python2或者python3均可,需要xlrd模块用于读取excel.

```sh
pip install xlrd
```

## 对excel表格的要求

可按两种模式来填写excel表格

### 模式1:
需要在excel中专门建一个名为xls2lua的Sheet,其中每一列对应一个需要转换的Sheet,列的第一行指定了Sheet到lua文件名的映射,其余行表示"数据列"到lua变量的映射,映射格式如下.
- '*'开头的映射名表示它在table中用作索引,索引可以有多个,但不能所有列都是索引.
- '#'结尾的映射名表示映射为数字.
- '$'结尾的映射名表示映射为字符串,指明了这种格式时,在excel中填写时无需加引号.
- '?'结尾的映射名表示映射为布尔变量,如果填的是字符串,会自动处理常见的值,比如(0, 1, 是,否...).
- 结尾不是"#$?"的,会尽可能将表格中的字面值照搬到lua中去,这时如果希望被处理为字符串的,需要在表格中自行添加引号.
具体参见test1.xls.

### 模式2:
无需在excel文件中加入额外的名为xls2lua的meta sheet,直接在列标题中标注即可;标注格式与方式1类似,具体示例参见test2.xls.
