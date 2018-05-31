# MOSAD 期中项目 实验报告

gyakkun part, @Team **ZgenY an Gyak**

----------------------------------------

目录

[TOC]


本项目所用到的UWP开发中的知识点

```text
报告要求: 

(2） 实验报告中建议写清楚如何运行，每个知识点对应如何实现的
```

```text
outline

(1) Adaptive UI					//全局Adaptive UI

(2) Data Binding				//全局Data Binding

(3) Database 					//本地化收藏夹存储

(4) App to app communication	//分享到SNS

(5) Network accessing 			//全程使用tmdb API

(6) File management				//可以将海报保存到本地

(7) Live tiles 					//收藏夹与动态磁贴联动

(8) 媒体应用					//bgm播放

```




## Adaptive UI - 全局Adaptive UI

大量采用相对布局, 如在MainPage.xaml中使用"RelativePanel"来定义汉堡界面按钮, 点击汉堡按钮时候会弹出展开的文字菜单。SplitView.Content中, 使用行自动宽高以实现行元素个数随窗口宽度变化。

```xaml
	<RelativePanel>
		<Button Name="HamburgerButton" 
			RelativePanel.AlignLeftWithPanel="True"
			FontFamily="Segoe MDL2 Assets"
			FontSize="36" 
			Content="&#xE700;" 
			Click="HamburgerButton_Click" />
```

```xaml
	<SplitView.Content>
		<Grid>
			<Grid.RowDefinitions>
				<RowDefinition Height="Auto"/>
				<RowDefinition Height="*"/>
				<RowDefinition Height="Auto"/>
			</Grid.RowDefinitions>
```



## Data Binding - 全局Data Binding

由于需要和服务器进行数据交互, 几乎每个需要展示到获取到的对象的地方都用到了数据绑定, 此处列出比较关键的部分。

MainPage部分, 使用Frame标签嵌入ListPage, 

```xaml
	</StackPanel>
	<Frame Name="ListFrame" Grid.Row="1" Navigated="ListFrame_Navigated"/>
```
```c#
	CurrentSourcePageType.Equals(typeof(ListPage))&&!ListFrame.CurrentSourcePageType.Equals(typeof(CollectorItems)) ? Visibility.Visible : Visibility.Collapsed;
```

而后在ListPage中, 定义数据模板, 对每个元素(电影项/TV项), 绑定从API获取来的title和poster_path：

```xaml
	<DataTemplate x:DataType="data:MovieResult" x:Key="MovieResultDataTemplate">
		<StackPanel HorizontalAlignment="Center">
			<Image Height="300" Source="{x:Bind poster_path}" Margin="5"/>
			<TextBlock FontSize="16" Margin="0,0,0,5" Text="{x:Bind title}" TextWrapping="Wrap" HorizontalAlignment="Center" />
		</StackPanel>
	</DataTemplate>
```
```xaml
	<DataTemplate x:DataType="data:TVResult" x:Key="TVResultDataTemplate">
		<StackPanel HorizontalAlignment="Center">
			<Image Height="300" Source="{x:Bind poster_path}" Margin="5"/>
			<TextBlock FontSize="16" Margin="0,0,0,5" Text="{x:Bind name}" TextWrapping="Wrap" HorizontalAlignment="Center" />
		</StackPanel>
	</DataTemplate>
```



## Database - 本地化收藏夹存储

新建一个CollectorItems.xaml, 专门负责收藏夹逻辑。由于数据持久化的需要, 在ViewModel中创建db实例, 使用SQLite来存储收藏夹项：

```c#

private ViewModel()
{
    SQLiteConnection db = App.conn;
    using (var statement = db.Prepare(App.SQL_QUERY_VALUE))
    {
        while (SQLiteResult.ROW == statement.Step())
        {

            Starlist.Add(new Star(Convert.ToInt32(statement[0]), (string)statement[1], (string)statement[2], (string)statement[3], (string)statement[4], Convert.ToInt32(statement[5])));

        }
    }

}
```

此处截取添加收藏夹项目的一小片段作为引例。

```c#
public void AddStar(Star st)
        {

            starlist.Add(st);
            TileService.GenerateTiles();
            var db = App.conn;

            try
            {
                using (var Item = db.Prepare(App.SQL_INSERT))
                {
                    Item.Bind(1, st.id);
                    Item.Bind(2, st.title);
                    Item.Bind(3, st.imagepath);
                    Item.Bind(4, st.posterpath);
                    Item.Bind(5, st.comment);
                    Item.Bind(6, st.type);
                    Item.Step();
                }
            }
```





















