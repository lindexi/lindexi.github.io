# matlab 画图

本文讲如何使用 matlab 画图。

本文包括：折线图的 x轴和y轴、标题、图例

柱状图填充图案

<!--more-->
<!-- csdn -->

## 折线图

折线图需要知道的是 Legend


```csharp
    legend(...,'Location',LOC) adds a legend in the specified
    location, LOC, with respect to the axes.  LOC may be either a
    1x4 position vector or one of the following strings:
        'North'              inside plot box near top
        'South'              inside bottom
        'East'               inside right
        'West'               inside left
        'NorthEast'          inside top right (default for 2-D plots)
        'NorthWest'          inside top left
        'SouthEast'          inside bottom right
        'SouthWest'          inside bottom left
        'NorthOutside'       outside plot box near top
        'SouthOutside'       outside bottom
        'EastOutside'        outside right
        'WestOutside'        outside left
        'NorthEastOutside'   outside top right (default for 3-D plots)
        'NorthWestOutside'   outside top left
        'SouthEastOutside'   outside bottom right
        'SouthWestOutside'   outside bottom left
        'Best'               least conflict with data in plot
        'BestOutside'        least unused space outside plot
```

## 直方图

如何画柱状图，如何在柱状图使用不同的图案填充？

matlab 柱状图填充图案可以使用 applyhatch 画图，下面将告诉大家如何做

第一步是复制下面代码到一个文件，这个文件名是`applyhatch.m`


```csharp
    function applyhatch(h,patterns,colorlist)
%APPLYHATCH Apply hatched patterns to a figure
%  APPLYHATCH(H,PATTERNS) creates a new figure from the figure H by
%  replacing distinct colors in H with the black and white
%  patterns in PATTERNS. The format for PATTERNS can be
%    a string of the characters '/', '\', '|', '-', '+', 'x', '.'
%    a cell array of matrices of zeros (white) and ones (black)
%
%  APPLYHATCH(H,PATTERNS,COLORS) maps the colors in the n by 3
%  matrix COLORS to PATTERNS. Each row of COLORS specifies an RGB
%  color value.
%
%  Note this function makes a bitmap image of H and so is limited
%  to low-resolution, bitmap output.
%
%  Example 1:
%    bar(rand(3,4));
%    applyhatch(gcf,'\-x.');
%
%  Example 2:
%    colormap(cool(6));
%    pie(rand(6,1));
%    legend('Jan','Feb','Mar','Apr','May','Jun');
%    applyhatch(gcf,'|-+.\/',cool(6));
%
%  See also: MAKEHATCH

%  By Ben Hinkle,
%  This code is in the public domain.

oldppmode = get(h,'paperpositionmode');
oldunits = get(h,'units');
set(h,'paperpositionmode','auto');
set(h,'units','pixels');
figsize = get(h,'position');
if nargin == 2
  colorlist = [];
end
bits = hardcopy(h,'-dzbuffer','-r0');
set(h,'paperpositionmode',oldppmode);

bwidth = size(bits,2);
bheight = size(bits,1);
bsize = bwidth * bheight;
if ~isempty(colorlist)
  colorlist = uint8(255*colorlist);
  [colors,colori] = nextnonbw(0,colorlist,bits);
else
  colors = (bits(:,:,1) ~= bits(:,:,2)) | ...
           (bits(:,:,1) ~= bits(:,:,3));
end
pati = 1;
colorind = find(colors);
while ~isempty(colorind)
  colorval(1) = bits(colorind(1));
  colorval(2) = bits(colorind(1)+bsize);
  colorval(3) = bits(colorind(1)+2*bsize);
  if iscell(patterns)
    pattern = patterns{pati};
  elseif isa(patterns,'char')
    pattern = makehatch(patterns(pati));
  else
    pattern = patterns;
  end
  pattern = uint8(255*(1-pattern));
  pheight = size(pattern,2);
  pwidth = size(pattern,1);
  ratioh = ceil(bheight/pheight);
  ratiow = ceil(bwidth/pwidth);
  bigpattern = repmat(pattern,[ratioh ratiow]);
  if ratioh*pheight > bheight
    bigpattern(bheight+1:end,:) = [];
  end
  if ratiow*pwidth > bwidth
    bigpattern(:,bwidth+1:end) = [];
  end
  bigpattern = repmat(bigpattern,[1 1 3]);
  color = (bits(:,:,1) == colorval(1)) & ...
          (bits(:,:,2) == colorval(2)) & ...
          (bits(:,:,3) == colorval(3));
  color = repmat(color,[1 1 3]);
  bits(color) = bigpattern(color);
  if ~isempty(colorlist)
    [colors,colori] = nextnonbw(colori,colorlist,bits);
  else
    colors = (bits(:,:,1) ~= bits(:,:,2)) | ...
             (bits(:,:,1) ~= bits(:,:,3));
  end
  colorind = find(colors);
  pati = (pati + 1);
  if pati > length(patterns)
    pati = 1;
  end
end

newfig = figure('units','pixels','visible','off');
imaxes = axes('parent',newfig,'units','pixels');
im = image(bits,'parent',imaxes);
fpos = get(newfig,'position');
set(newfig,'position',[fpos(1:2) figsize(3) figsize(4)+1]);
set(imaxes,'position',[0 0 figsize(3) figsize(4)+1],'visible','off');
set(newfig,'visible','on');

function [colors,out] = nextnonbw(ind,colorlist,bits)
out = ind+1;
colors = [];
while out <= size(colorlist,1)
  if isequal(colorlist(out,:),[255 255 255]) | ...
        isequal(colorlist(out,:),[0 0 0])
    out = out+1;
  else
    colors = (colorlist(out,1) == bits(:,:,1)) & ...
             (colorlist(out,2) == bits(:,:,2)) & ...
             (colorlist(out,3) == bits(:,:,3));
    return
  end
end

% 而applyhatch函数需要调用下面的函数

function A = makehatch(hatch)
%MAKEHATCH Predefined hatch patterns
%  MAKEHATCH(HATCH) returns a matrix with the hatch pattern for HATCH
%   according to the following table:
%      HATCH        pattern
%     -------      ---------
%        /          right-slanted lines
%        \          left-slanted lines
%        |          vertical lines
%        -          horizontal lines
%        +          crossing vertical and horizontal lines
%        x          criss-crossing lines
%        .          single dots
%
%  See also: APPLYHATCH

%  By Ben Hinkle,
%  This code is in the public domain.

n = 6;
A=zeros(n);
switch (hatch)
case '/'
  A = fliplr(eye(n));
case '\'
  A = eye(n);
case '|'
  A(:,1) = 1;
case '-'
  A(1,:) = 1;
case '+'
  A(:,1) = 1;
  A(1,:) = 1;
case 'x'
  A = eye(n) | fliplr(diag(ones(n-1,1),-1));
case '.'
  A(1:2,1:2)=1;
otherwise
  error(['Undefined hatch pattern "' hatch '".']);
end

```
第二步是把文件拷贝到工作台，工作台是什么，就是软件打开的文件夹路径，这个路径如下图

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201751094129.jpg)

可以自己修改，只要把上面的文件放在这个文件夹就可以了。

使用方式是`applyhatch(gcf,'\.x.');`

其中第二个参数就是使用不同的图案，可以使用添加`'/', '\', '|', '-', '+', 'x', '.'`几个字符

例子：


```csharp
    close all
clear all
clc
% The computer journal paper 
% Jigang Wu etc.
% copyright: lonchen@mail.ustc.edu.cn


% data = [31,32,35;72,73,75;113,114,117;144,146,147;171,173,174;213,215,220];
data =[16,17,18;33,34,37;51,54,55;71,74,74;86,91,91;105,113,113];
bar(data,1);
axis([0 7 0.0 150]);
legend('DPA','TSRP','GRP',0);
grid on;
set(gca,'XTickLabel',{'100','200','300','400','500','600'});

xlabel('The number of internal nodes');
ylabel('The number of replica that created');
set(gcf,'Color','w'); % 设置背景颜色为白色,否则坐标轴出现区域的颜色为灰色
applyhatch(gcf,'\.x.');


```

对于不同组合的直方图，使用 `data=[数据1.1，数据1.2，数据1.3；数据2.1，数据2.2……]`

然后画出来，使用`bar(data,1);` 第二个参数是宽度，自己尝试修改第二个值跑一下。

可以使用图例，matlab的图例使用的`legend('DPA','TSRP','GRP',0);` 有多少个数据就添加对应图例。

试试下面代码：


```csharp
    y=[559006 ,2269384,783762;508559 ,2140905,696001;506491,2007763,735464]
bar(y,0.6)

legend('n','N','l') 
grid on;
set(gca,'XTickLabel',{'第一次','第二次','第三次'}) 

xlabel('匹配次数')

ylabel('结果数')

set (gcf,'Position',[500,500,500,500], 'color','w') 

applyhatch(gcf,'\.x./');


set(gcf,'Color','w'); 
```

![](http://7xqpl8.com1.z0.glb.clouddn.com/AwCCAwMAItoFADbzBgABAAQArj4BAGZDAgBo6AkA6Nk%3D%2F201751094956.jpg)

如果过程遇到问题，可以联系我 lindexi_gd@163.com。

感谢陈龙师兄的帮助。

<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/"><img alt="知识共享许可协议" style="border-width:0" src="https://i.creativecommons.org/l/by-nc-sa/4.0/88x31.png" /></a><br />本作品采用<a rel="license" href="http://creativecommons.org/licenses/by-nc-sa/4.0/">知识共享署名-非商业性使用-相同方式共享 4.0 国际许可协议</a>进行许可。欢迎转载、使用、重新发布，但务必保留文章署名[林德熙](http://blog.csdn.net/lindexi_gd)(包含链接:http://blog.csdn.net/lindexi_gd )，不得用于商业目的，基于本文修改后的作品务必以相同的许可发布。如有任何疑问，请与我[联系](mailto:lindexi_gd@163.com)。 
