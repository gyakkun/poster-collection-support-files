# MOSAD 期中项目 实验报告

gyakkun part, @Team **ZgenY an Gyak**

----------------------------------------

**目录**

[TOC]

## Overview - 概览

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
<!-- MainPage.xaml -->
<RelativePanel>
		<Button Name="HamburgerButton" 
			RelativePanel.AlignLeftWithPanel="True"
			FontFamily="Segoe MDL2 Assets"
			FontSize="36" 
			Content="&#xE700;" 
			Click="HamburgerButton_Click" />
```



```xaml
<!-- MainPage.xaml -->
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
<!-- MainPage.xaml -->
</StackPanel>
	<Frame Name="ListFrame" Grid.Row="1" Navigated="ListFrame_Navigated"/>
```
在App.xaml.cs中定义OnNavigated方法, 控制ListPage的显示：

```c#
//App.xaml.cs
CurrentSourcePageType.Equals(typeof(ListPage))&&!ListFrame.CurrentSourcePageType.Equals(typeof(CollectorItems)) ? Visibility.Visible : Visibility.Collapsed;
```

而后在ListPage中, 定义数据模板, 对每个元素(电影项/TV项), 绑定从API获取来的title和poster_path：

```xaml
<!-- ListPage.xaml -->
<DataTemplate x:DataType="data:MovieResult" x:Key="MovieResultDataTemplate">
		<StackPanel HorizontalAlignment="Center">
			<Image Height="300" Source="{x:Bind poster_path}" Margin="5"/>
			<TextBlock FontSize="16" Margin="0,0,0,5" Text="{x:Bind title}" TextWrapping="Wrap" HorizontalAlignment="Center" />
		</StackPanel>
	</DataTemplate>
```
```xaml
<!-- ListPage.xaml -->
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
//ViewModel.cs
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
//ViewModel.cs
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



 ## App to app communication - 利用系统分享功能

在详情页(*DetailPage.xaml*)以及独立的海报展示页(*ShowPosterPage.xaml.cs*)加入分享按钮(找不到合适的图标, 遂使用了"people"), 并完成逻辑, 实现分享图片和剧情简介给好友的功能。实测可以通过Win10自带邮件应用和QQ完成分享。

```xaml
<!-- ShowPosterPage.xaml -->
<Page.BottomAppBar>
        <CommandBar>
            <AppBarButton
                x:Name="shareWithFriends"
                Icon="People"
                Label="AppBarButton"
                Click="shareWithFriends_Click" />
            <AppBarButton Name="savePictureAppBarButton" Icon="Save" Label="Save" Click="savePictureAppBarButton_Click"/>
            
        </CommandBar>
```

逻辑:

```c#
//ShowPosterPage.xaml.cs
private void OnShareDataRequested(DataTransferManager sender, DataRequestedEventArgs args)
        {

            DataRequest request = args.Request;

            var deferral = args.Request.GetDeferral();
            try {
                request.Data.Properties.Title = "Share";
                request.Data.Properties.Description = "Share this picture.";
                request.Data.SetText(url);
                //添加图片
                RandomAccessStreamReference imageRASR = RandomAccessStreamReference.CreateFromUri(((BitmapImage)image.Source).UriSource);
                // 建议用Thumbnail分享文件
                request.Data.Properties.Thumbnail = imageRASR;
                request.Data.SetBitmap(imageRASR);
            }
            finally
            {
                deferral.Complete();
            }
        }
        private void shareWithFriends_Click(object sender, RoutedEventArgs e)
        {
            var s = sender as FrameworkElement;
            DataTransferManager.ShowShareUI();
        }
```



 ## Network accessing - 全程使用TMDB API

本次UWP其中项目的核心便是获取电影海报数据。在之前的Web课外实践中, 得知了TMDB这个网站以及其较完善的API, 遂采用其作为本项目的核心API。但在后期的开发与使用中逐渐发现**学校网络对该国外API的访问性能并不太好**。经测试人员反馈, 使用杭州电信, 广州花都区移动, 广州电信, 以及美国国内的线路访问速度较快。**TA若使用时出现加载缓慢的现象, **除开敝组在网络优化上存在不足以外, **校园网质量欠佳也是重要的原因之一。**

使用API大量涉及异步方法, 这也是网络编程的核心之一。C#提供了原生的await/async方法库, 使得网络编程变得简单, 比javascript高到不知哪里去了(笑)。

由于通篇都使用到了网络访问, 所以下面会列举出两处核心的调用作为例子。

首页初始化过程, 访问tmdb主页获取推荐列表:

```c#
//MainPage.xaml.cs 
if (VideoTypeComboBox.SelectedIndex == 0)
                {
                    flag = 0;
                    String url = String.Format("https://api.themoviedb.org/3/discover/movie?api_key=7888f0042a366f63289ff571b68b7ce0&include_adult=false{0}&page={1}{2}{3}{4}", language,page,Mgenre,releaseYear,sortBy);
                    HttpClient client = new HttpClient();
                    String Jresult = await client.GetStringAsync(url);
                    DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(QueryMovieList));
                    MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(Jresult));
                    QueryMovieList queryMovieList = (QueryMovieList)serializer.ReadObject(ms);
                    if (queryMovieList.total_results == 0)
                    {
                        await new Windows.UI.Popups.MessageDialog("Found nothing, please change the key words and try again! ").ShowAsync();
                    }
                    else
                    {
                        viewModel.clear();
                        foreach (var result in queryMovieList.results)
                        {
                            if (result.poster_path != null)
                            {
                                result.poster_path = "https://image.tmdb.org/t/p/w500" + result.poster_path;
                            }
                            else
                            {
                                result.poster_path = "Assets/defaultPoster.jpg";
                            }
                            viewModel.AddMovieResult(result);
                        }
                    }
```

列表详情页, 点进相应的电影项, 需要获取电影详情

```c#
//ListPage.xaml.cs
private async void GridView_MovieItemClick(object sender, ItemClickEventArgs e)
        {
            try
            {
                var item = (MovieResult)e.ClickedItem;
                String url = String.Format("https://api.themoviedb.org/3/movie/{0}?api_key=7888f0042a366f63289ff571b68b7ce0&append_to_response=casts", item.id);
                HttpClient client = new HttpClient();
                String Jresult = await client.GetStringAsync(url);
                DataContractJsonSerializer serializer = new DataContractJsonSerializer(typeof(MovieDetail));
                MemoryStream ms = new MemoryStream(Encoding.UTF8.GetBytes(Jresult));
                viewModel.TheMovieDetail = (MovieDetail)serializer.ReadObject(ms);

                if (viewModel.TheMovieDetail.backdrop_path != null)
                {
                    viewModel.TheMovieDetail.backdrop_path = "https://image.tmdb.org/t/p/original" + viewModel.TheMovieDetail.backdrop_path;
                }
                else
                {
                    viewModel.TheMovieDetail.backdrop_path = "Assets/defaultBackground.png";
                }
                if (viewModel.TheMovieDetail.poster_path != null)
                {
                    viewModel.TheMovieDetail.poster_path = "https://image.tmdb.org/t/p/w500" + viewModel.TheMovieDetail.poster_path;
                }
                else
                {
                    viewModel.TheMovieDetail.poster_path = "Assets/defaultPoster.jpg";
                }

                foreach (var cast in viewModel.TheMovieDetail.casts.cast)
                {
                    if (cast.profile_path != null)
                    {
                        cast.profile_path = "https://image.tmdb.org/t/p/w500" + cast.profile_path;
                    }
                    else
                    {
                        cast.profile_path = "Assets/defaultPhoto.jpg";
                    }
                }

                this.Frame.Navigate(typeof(DetailPage), 0);
            }
            catch
            {
                await new Windows.UI.Popups.MessageDialog("Opps! This item cannot be serialized, please try another item! ").ShowAsync();
            }
            
        }
```



 ## File management

 ## Live tiles

 ## Media


















