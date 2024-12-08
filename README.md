
在系统的权限管理中，往往都会涉及到用户的选择处理，特别是基于角色的访问控制中，很多情况下需要用到选择用户的处理。本篇随笔，基于WxPython跨平台开发框架，采用原有开发框架成熟的一套权限系统理念，对机构、用户、角色、权限、菜单、日志、字典等内容进行管理的，因此也涉及到了用户选择的处理，在WxPython开发中，为了方便，我们往往会构建一些自定义控件，以便重用处理，本篇设计了标签组件来简化一些处理操作，同时可以在很多地方进行重用。


### 1、权限内容的相关介绍


RBAC（Role\-Based Access Control，基于角色的访问控制）是一种常用的访问控制模型，在此模型中，系统中的每个用户被赋予一个或多个角色，权限则与角色相关联。角色是一组权限的集合，用户通过其所扮演的角色来获得相应的权限，从而限制用户对系统资源的访问。


#### 基本概念


1. **角色（Role）**： 角色是一个标识符，表示一组权限的集合。每个角色可以有不同的权限，例如 "管理员"、"普通用户"、"审计员" 等。
2. **用户（User）**： 用户是系统中的实际操作主体。每个用户可以分配一个或多个角色，根据角色来获得相应的权限。
3. **权限（Permission）**： 权限是指用户可以执行的操作或可以访问的资源。例如，"读取文件"、"修改文件"、"删除文件" 等。
4. **会话（Session）**： 会话是用户和系统之间交互的时间段，系统通过会话信息管理用户的角色和权限。


#### RBAC的工作原理


RBAC模型的基本思想是：


* 用户通过其角色获得访问权限。
* 系统将权限与角色进行绑定，角色分配给用户。
* 用户通过角色来执行操作，而不是直接赋予权限。


#### RBAC的主要组成部分


1. **角色权限映射**： 角色和权限之间的映射关系是RBAC的核心。例如，"管理员"角色拥有创建、删除、编辑用户的权限，而"普通用户"角色只拥有读取数据的权限。
2. **用户角色映射**： 用户通过角色来获得权限。例如，某个用户被赋予"经理"角色，那么该用户便获得了与"经理"角色相关的所有权限。
3. **访问控制**： 系统根据用户当前角色的权限来判断是否允许访问某个资源或执行某个操作。


#### RBAC的实现


1. **定义角色和权限**： 在系统中定义不同的角色，例如管理员、用户、访客等，并为每个角色分配不同的权限。
2. **用户分配角色**： 每个用户通过管理员或系统自动分配角色，获得该角色所拥有的权限。
3. **控制访问**： 系统根据用户的角色判断是否允许其访问某些资源或执行某些操作。


我曾经在《[Winform开发框架之权限管理系统改进的经验总结（2）\-用户选择界面的设计](https://github.com/wuhuacong/p/3493137.html "发布于 2013-12-26 22:15")》中介绍了选择用户的界面，


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241207170151931-456646079.png)


 可以在组织机构或者用户角色管理中选择相关用户，我们在WxPython跨平台开发框架同样也是采用这种管理概念。界面设计效果如下所示。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241207170750476-1347159369.png)


弹出的界面中，模仿了之前的用户界面，由于之前在添加每个用户的时候，采用的是添加自定义控件的方式呈现，在wxpython中做一些自定义控件的处理也是非常方便的，我们先来看最终的界面效果，后面在分析如何实现用户标签的添加和移除等操作管理。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241207170422456-1634341875.png)


### 2、标签信息的自定义控件的处理


为了实现标签信息的处理，我们先来做一个简单的标签组件的测试，在整合在界面中进行完善的处理。


简单的例子代码如下所示。




```
import wx
import random


class Tag(wx.Panel):
    def __init__(self, parent, label, on_close):
        """标签控件初始化
        :param parent: 父控件
        :param label: 标签文本
        :param on_close: 关闭事件回调函数
        """
        super().__init__(parent, style=wx.BORDER_SIMPLE)

        self.on_close = on_close  # 回调函数，用于处理关闭事件
        self.label = label

        # 设置背景颜色，方便区分标签
        # self.SetBackgroundColour(wx.Colour(240, 240, 240))
        r = random.randint(128, 255)
        g = random.randint(128, 255)
        b = random.randint(128, 255)
        self.SetBackgroundColour(wx.Colour(r,g,b))

        # 创建控件
        self.text = wx.StaticText(self, label=label)
        self.close_btn = wx.Button(self, label="x", size=(20, 20))

        # 布局
        sizer = wx.BoxSizer(wx.HORIZONTAL)
        sizer.Add(self.text, 0, wx.ALIGN_CENTER | wx.RIGHT, 5)
        sizer.AddSpacer(2)
        sizer.Add(self.close_btn, 0, wx.ALIGN_CENTER)

        self.SetSizer(sizer)
        sizer.Fit(self)

        # 绑定事件
        self.close_btn.Bind(wx.EVT_BUTTON, self._on_close)

    def _on_close(self, event):
        """处理关闭事件"""
        if self.on_close:
            self.on_close(self)


class TagPanel(wx.Panel):
    def __init__(self, parent):
        """标签面板控件初始化
        :param parent: 父控件
        """
        super().__init__(parent)

        self.sizer = wx.WrapSizer(wx.HORIZONTAL, wx.WRAPSIZER_DEFAULT_FLAGS)
        self.SetSizer(self.sizer)

        # 测试用标签
        for i in range(5):
            self.add_tag(f"张三发 {i+1}")

    def add_tag(self, label):
        """添加一个新标签"""
        tag = Tag(self, label, self.remove_tag)
        self.sizer.Add(tag, 0, wx.ALL, 5)
        self.Layout()

    def remove_tag(self, tag):
        """移除一个标签"""
        self.sizer.Detach(tag)
        tag.Destroy()
        self.Layout()


class MyFrame(wx.Frame):
    def __init__(self):
        super().__init__(None, title="标签测试案例", size=(400, 300))

        panel = wx.Panel(self)
        main_sizer = wx.BoxSizer(wx.VERTICAL)

        self.tag_panel = TagPanel(panel)
        main_sizer.Add(self.tag_panel, 1, wx.EXPAND | wx.ALL, 10)

        # 添加标签的按钮
        add_button = wx.Button(panel, label="添加新标签")
        add_button.Bind(wx.EVT_BUTTON, self.on_add_tag)
        main_sizer.Add(add_button, 0, wx.ALIGN_CENTER | wx.ALL, 10)

        panel.SetSizer(main_sizer)

    def on_add_tag(self, event):
        """添加新标签"""
        self.tag_panel.add_tag(f"标签 {len(self.tag_panel.sizer.GetChildren()) + 1}")

if __name__ == "__main__":
    app = wx.App()
    frame = MyFrame()
    frame.Show()
    app.MainLoop()
```


实现的效果如下所示。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241207173708919-898042491.png)


大概就是这样的效果，我们在添加用户的界面中整合它，并完善一些操作，如可以记录选中的相关信息，清空记录，可以更新选中的记录数等信息。


首先，我们增加一个通用的对象类，来承载显示内容和值信息的（text,value键值对)，如下定义所示。




```
class CListItem(BaseModel):
    """框架用来记录字典键值的类，用于Comobox等控件对象的值传递"""

    def __init__(self, text: str, value: Any, **data: Any):
        super().__init__(**data)
        self.text = text
        self.value = value

    text: str = None
    value: Any = None

    def __repr__(self):
        return f"CListItem(text={self.text}, value={self.value})"
```


因此Tag的类初始化，改用该对象来来处理，如下代码所示




```
class MyTag(wx.Panel):
    """自定义标签, 带有关闭按钮"""

    item: CListItem = None  # 标签数据

    def __init__(self, parent, item: CListItem, on_close):
        """初始化

        :param parent: 父容器
        :param label: 标签文本
        :param on_close: 关闭事件回调函数
        """
        super().__init__(parent, style=wx.BORDER_SIMPLE)

        self.on_close = on_close  # 回调函数，用于处理关闭事件
        self.item = item

        # 设置背景颜色，方便区分标签
        # self.SetBackgroundColour(wx.Colour(240, 240, 240))
        r = random.randint(128, 255)
        g = random.randint(128, 255)
        b = random.randint(128, 255)
        self.SetBackgroundColour(wx.Colour(r, g, b))

        # 创建控件
        self.text = wx.StaticText(self, label=item.text)
        self.close_btn = wx.Button(self, label="x", size=(20, 20))

        # 布局
        sizer = wx.BoxSizer(wx.HORIZONTAL)
        sizer.Add(self.text, 0, wx.ALIGN_CENTER | wx.RIGHT, 5)
        sizer.AddSpacer(2)
        sizer.Add(self.close_btn, 0, wx.ALIGN_CENTER)

        self.SetSizer(sizer)
        sizer.Fit(self)

        # 绑定事件
        self.close_btn.Bind(wx.EVT_BUTTON, self._on_close)

    def _on_close(self, event):
        """处理关闭事件"""
        if self.on_close:
            self.on_close(self)
```


标签面板的内容中，我们也改用 wx.WrapSizer 这个可折叠的面板Sizer来做标签控件的展示，这样可以在一个位置上堆叠，并且可以换行处理等，比较常规的Panel会好一些。


同时，标签面板增加一个更新的处理函数给外部定义，并且在添加的时候，对重复添加的进行判断，这样我们在标签的添加、删除等操作都进行外部事件的调用就完美了。完整的代码如下所示。




```
class MyTagPanel(wx.Panel):
    def __init__(
        self,
        parent,
        id=wx.ID_ANY,
        pos=wx.DefaultPosition,
        size=wx.DefaultSize,
        on_tag_updated=None,
        *args,
        **kwargs,
    ):
        super().__init__(parent, id, pos, size, *args, **kwargs)

        self.sizer = wx.WrapSizer(wx.HORIZONTAL, wx.WRAPSIZER_DEFAULT_FLAGS)
        self.SetSizer(self.sizer)

        self.list = []  # 标签数据列表
        self.on_tag_updated = on_tag_updated  # 回调函数，用于处理标签变化事件

    def _find_tag(self, item: CListItem):
        """查找标签"""
        # for child in self.list:
        #     if item.value == child.value:
        #         return item
        # return None

        # next 函数：用于从迭代器中获取第一个匹配的元素。如果找不到匹配项，可以指定默认值（这里为 None）。
        return next((child for child in self.list if item.value == child.value), None)

    def add_tag(self, item: CListItem):
        """添加一个新标签"""
        if self._find_tag(item):
            return  # 标签已存在，不再添加
        self.list.append(item)

        tag = MyTag(self, item, self.remove_tag)
        self.sizer.Add(tag, 0, wx.ALL, 5)
        self.Layout()

        self.update_tag()

    def update_tag(self):
        # 触发回调函数
        if self.on_tag_updated:
            self.on_tag_updated()

    def remove_tag(self, tag: MyTag):
        """移除一个标签"""

        self.list.remove(tag.item)  # 从列表中移除标签数据

        self.sizer.Detach(tag)
        tag.Destroy()
        self.Layout()

        self.update_tag()

    def get_items(self):
        """获取标签列表"""
        items = []
        for child in self.sizer.GetChildren():
            tag = child.GetWindow()
            tag: MyTag  # 限定类型
            items.append(tag.item)
        return items

    def clear_items(self):
        """清空标签列表"""
        for child in self.sizer.GetChildren():
            tag = child.GetWindow()
            tag: MyTag  # 限定类型
            self.remove_tag(tag)

        self.update_tag()
```


### 3、在用户选择界面中使用标签控件和标签面板


我们来看看在选择用户的界面中添加标签面板和相关处理的按钮，如下界面效果所示。


![](https://img2024.cnblogs.com/blog/8867/202412/8867-20241207175428092-1248149143.png)


 添加标签面板我们用一个函数来处理，如下所示。




```
    def create_tag_panel_buttons(self, parent, style=wx.BORDER_SIMPLE):
        """创建标签面板和操作按钮"""
        dlg_sizer: wx.BoxSizer = self.GetSizer()

        # 添加选择区标签面板
        self.tag_panel = ctrl.MyTagPanel(
            parent, size=(-1, 100), style=style, on_tag_updated=self.OnTagsUpdated
        )
        dlg_sizer.Add(self.tag_panel, 0, wx.EXPAND | wx.ALL, 10)

        # 添加按钮区
        btn_sizer = self.CreateCustomButtons(self)
        dlg_sizer.Add(
            btn_sizer, 0, wx.EXPAND | wx.BOTTOM | wx.RIGHT, self.GetDialogBorder()
        )
```


我们在标签的变化处理事件OnTagsUpdated实现通知外部标签显示的处理，如下代码所示。




```
    def OnTagsUpdated(self):
        """更新选择的用户信息"""
        items = self.tag_panel.get_items()
        self.lbl_selected_text.SetLabel(f"已选择【{len(items)}】个项目")
```


我们在表格的双击事件或者按钮事件中添加对应的用户到面板中，我们获得记录的id和用户姓名添加到标签面板即可。




```
            # 获取该行的主键值和值
            entity_id = self.table.GetPrimaryKeyValue(row)
            fullname = self.table.GetValueByKey(row, "fullname")
            if entity_id and fullname:
                self.tag_panel.add_tag(CListItem(fullname, entity_id))
            else:
                MessageUtil.show_info(self, "该行无主键值 或 真实姓名")
```


如果我们需要再外部清空，也是调用标签面板的清空函数，如果需要获得选中的记录，也还是调用标签面板的选择记录集合即可。


如确认选择的处理事件中，代码如下所示。




```
    async def OnConfirmSelect(self, event):
        """确认选择"""
        items = self.tag_panel.get_items()
        if len(items) == 0:
            MessageUtil.show_warning(self, "请先选择项目")
            return

        self.selected_items = items  # 保存选择的记录
        self.CloseModel(wx.ID_OK)
```


因此简单的封装一下，就可以实现了标签记录的相关处理，非常方便，当前其他日常的功能实现，也是如此简化即可。


当然，对于选择用户的整个界面窗体，我们也是可以把它当做一个公用组件来实现的，因此在机构和角色中都需要对用户进行关联，因此它们公用一个界面处理。


以上就是对于常规界面中常用到的一些功能，进行了相关的封装，然后在系统中整合使用的整个过程。


 本博客参考[蓝猫机场](https://fenfang.org)。转载请注明出处！
