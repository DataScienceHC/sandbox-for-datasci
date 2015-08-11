
<div style="text-align: center"><img src="https://db.tt/NpW3noem"></div>

# R code with pipes
這場[DSHC meetup](http://www.meetup.com/Data-Science-HC/events/224525514/)的講題是R code with pipes，pipes是一種讓程式可讀性更高的寫作style，這場talk將展示如何透過magrittr套件，進化R語言寫作的習慣。

<div style="text-align: center"><img src="https://upload.wikimedia.org/wikipedia/en/b/b9/MagrittePipe.jpg"></div>
<center>Ceci n’est pas une pipe (French for "This is not a pipe") made by the Belgian surrealist painter René Magritte</center>

# magrittr 簡介
- Simplifying R code with pipes
- 2014年最有影響力的R套件之一
- 展開的程式碼會產生很多暫存變數
- 壓縮的程式碼不好讀
- 套件magrittr部份解決了這個問題
- 基本算子 `%>%`, `%<>%`, `%T>%`, `%$%`

# 1. 基本算子 (`%>%`)
- 想像一下程式的寫作與閱讀邏輯
- `%>%` 會將算子左邊的物件 (object) 傳到右邊的函數 (function) 中第一個argument
- `.` 點號適合用在欲傳入變數不是在傳入函數的第一個位置時使用
- use `x %>% f`, rather than `f(x)`
- or use `x %>% f(y, z)`, rather than `f(x, y, z)`
- or `y %>% f(x, ., z)`, rather than `f(x, y, z)`

## 1.1. 第一個例子


    # install.packages("magrittr")
    library(magrittr)
    x <- 1:10
    
    mean(x)
    x %>% mean # 由左而右順序操作




5.5






5.5




    # How it work
    "%>%" <- function(x,f) do.call(f,list(x)) # see http://stackoverflow.com/a/8897778
    pi %>% sin %>% cos  # 1

## 1.2. 幾種等價用法
- 利用三角形面積公式說明`%>%`算子的幾種等價用法


    tri_area <- function(a, h=5) a*h/2
    a <- 10
    tri_area(a)
    a %>% tri_area          # 省略括號
    a %>% tri_area(h=5)     # 保留括號
    a %>% tri_area(., h=5)  # 以 `.` 來表示欲傳入的變數




25






25






25






25



## 1.3. 複雜一點
感受一下指令壓縮的簡便與痛苦，你知道以下指令的意義嗎？


    a_list <- list(1:4, 3:5, 4:7)
    lapply(a_list, function(x) setdiff(sort(unique(unlist(a_list))), x))

把指令分解後，試著解讀看看


    a_list <- list(1:4, 3:5, 4:7)
    unique_item <- sort(unique(unlist(a_list))) # 1 2 3 4 5 6 7
    lapply(a_list, function(x) setdiff(unique_item, x))




<ol>
	<li><ol class=list-inline>
	<li>5</li>
	<li>6</li>
	<li>7</li>
</ol>
</li>
	<li><ol class=list-inline>
	<li>1</li>
	<li>2</li>
	<li>6</li>
	<li>7</li>
</ol>
</li>
	<li><ol class=list-inline>
	<li>1</li>
	<li>2</li>
	<li>3</li>
</ol>
</li>
</ol>




用 pipe style 後，code是不是比較好"讀"?


    a_list %>% unlist %>% unique %>% sort %>% {
       lapply(a_list, setdiff, x = .)
    }




<ol>
	<li><ol class=list-inline>
	<li>5</li>
	<li>6</li>
	<li>7</li>
</ol>
</li>
	<li><ol class=list-inline>
	<li>1</li>
	<li>2</li>
	<li>6</li>
	<li>7</li>
</ol>
</li>
	<li><ol class=list-inline>
	<li>1</li>
	<li>2</li>
	<li>3</li>
</ol>
</li>
</ol>




## 1.4. 創建一元函數


    f <- . %>% sum(.) %>% sqrt
    # is equivalent to 
    f <- function(.) sqrt(sum(.))
    f(1:10)




7.41619848709566



## 1.5. 大括號 `{ }` 與 點號 `.` 的用法
- 括號裡面的只要不是其他 `%>%` 後面的`.` 都代表你前面傳入的值
- `x %>% f(y = nrow(.), z = ncol(.))` is equivalent to `f(x, y = nrow(x), z = ncol(x))`
- `x %>% {f(y = nrow(.), z = ncol(.))}` is equivalent to `f(y = nrow(x), z = ncol(x))`


    f <- function(x=NA, y=NA, z=NA) {
      cat("x =", x, "\n")
      cat("y =", y, "\n")
      cat("z =", z, "\n\n")
    }
    
    1:10 %>% f(min(.), max(.))   # f(x=1:10, y=min(1:10), y=max(1:10))
    
    1:10 %>% f(y=min(.), z=max(.))   # f(x=1:10, y=min(1:10), y=max(1:10))
    
    1:10 %>% {f(y=min(.), z=max(.))} # f(x=NA, y=min(1:10), z=max(1:10))

    x = 1 2 3 4 5 6 7 8 9 10 
    y = 1 
    z = 10 
    
    x = 1 2 3 4 5 6 7 8 9 10 
    y = 1 
    z = 10 
    
    x = NA 
    y = 1 
    z = 10 
    


## 1.6. 再一個常見的例子


    plot(density(sample(mtcars$mpg, size=10000, replace=TRUE), kernel="gaussian"), col="red", main="density of mpg")


    mtcars$mpg %>% 
        sample(size=10000, replace=TRUE) %>% 
        density(kernel="gaussian") %>% 
        plot(col="red", main="density of mpg")



# 2. 其他算子
## 2.1. 具有賦值功能的 `%<>%` 算子
不僅計算新值 (`>`)，同時改變傳入物件的值 (`<`)，但不會輸出(`print`)結果


    b <- 9
    
    b %<>% sqrt
    b
    # 等價於以下兩種寫法
    # b %<>% sqrt %>% print 
    # b <- b %>% sqrt %>% print




3



## 2.2. 只傳遞，不回傳值的 %T>% 算子 (tee operations)
- 大多用於不傳回值的函數，譬如：`plot`
- 或是程式寫到一個段落，需要暫時斷開pipe line的時候


    women %T>%
        plot %>% # plot usually does not return anything.
        colMeans




<dl class=dl-horizontal>
	<dt>height</dt>
		<dd>65</dd>
	<dt>weight</dt>
		<dd>136.733333333333</dd>
</dl>






## 2.3. 可以取得變數的 `%$%` 算子 (exposition operations)
- 直接把前面物件的特定變數直接叫出來
- 某些情況下，欲傳遞的函數不需要接受左邊物件傳過來的所有資訊 (只需要其中幾欄)時適用


    iris %T>% 
        plot(Petal.Width~Sepal.Length, data=.) %$%
        cor(x=Sepal.Length, y=Petal.Width)




0.817941126271575





- Pipe 算子右邊的計算內容可以被包在大括號 {} 中一起執行


    fit <- lm(Petal.Width ~ Sepal.Length, data=iris)
    sigma_hat <- fit %$% {crossprod(residuals) / df.residual}
    sigma_hat




<table>
<tbody>
	<tr><td>0.1935963</td></tr>
</tbody>
</table>




## 2.4. `->` 算子


    which(1:10 > 5) -> id
    id




<ol class=list-inline>
	<li>6</li>
	<li>7</li>
	<li>8</li>
	<li>9</li>
	<li>10</li>
</ol>





    iris %>% lm(Petal.Width ~ Sepal.Length, data=.) %$%
        {crossprod(residuals) / df.residual} -> sigma_hat2

## 2.5 別名 (Aliases) 


    1:10 %>% add(1) 
    # is equivalent to
    # 1:10 + 1
    
    1:10 %>% multiply_by(2)
    # is equivalent to
    # 1:10 * 2




<ol class=list-inline>
	<li>2</li>
	<li>3</li>
	<li>4</li>
	<li>5</li>
	<li>6</li>
	<li>7</li>
	<li>8</li>
	<li>9</li>
	<li>10</li>
	<li>11</li>
</ol>







<ol class=list-inline>
	<li>2</li>
	<li>4</li>
	<li>6</li>
	<li>8</li>
	<li>10</li>
	<li>12</li>
	<li>14</li>
	<li>16</li>
	<li>18</li>
	<li>20</li>
</ol>





    iris %>%
       extract(, 1:4) %>%
       head
    # is equivalent to 
    # iris[,1:4] %>% head




<table>
<thead><tr><th></th><th scope=col>Sepal.Length</th><th scope=col>Sepal.Width</th><th scope=col>Petal.Length</th><th scope=col>Petal.Width</th></tr></thead>
<tbody>
	<tr><th scope=row>1</th><td>5.1</td><td>3.5</td><td>1.4</td><td>0.2</td></tr>
	<tr><th scope=row>2</th><td>4.9</td><td>3</td><td>1.4</td><td>0.2</td></tr>
	<tr><th scope=row>3</th><td>4.7</td><td>3.2</td><td>1.3</td><td>0.2</td></tr>
	<tr><th scope=row>4</th><td>4.6</td><td>3.1</td><td>1.5</td><td>0.2</td></tr>
	<tr><th scope=row>5</th><td>5</td><td>3.6</td><td>1.4</td><td>0.2</td></tr>
	<tr><th scope=row>6</th><td>5.4</td><td>3.9</td><td>1.7</td><td>0.4</td></tr>
</tbody>
</table>




## 目前magrittr (version 1.5.0) 支援的別名有：

<table>
<tr>
<td> extract </td>
<td> `[` </td>
</tr>
<tr>
<td> extract2 </td>
<td> `[[` </td>
</tr>
<tr>
<td> inset </td>
<td> `[<-` </td>
</tr>
<tr>
<td> inset2 </td>
<td> `[[<-` </td>
</tr>
<tr>
<td> use_series </td>
<td> `$` </td>
</tr>
<tr>
<td> add </td>
<td> `+` </td>
</tr>
<tr>
<td> subtract </td>
<td> `-` </td>
</tr>
<tr>
<td> multiply_by </td>
<td> `*` </td>
</tr>
<tr>
<td> raise_to_power </td>
<td> `^` </td>
</tr>
<tr>
<td> multiply_by_matrix </td>
<td> `%*%` </td>
</tr>
<tr>
<td> divide_by </td>
<td> `/` </td>
</tr>
<tr>
<td> divide_by_int </td>
<td> `%/%` </td>
</tr>
<tr>
<td> mod </td>
<td> `%%` </td>
</tr>
<tr>
<td> is_in </td>
<td> `%in%` </td>
</tr>
<tr>
<td> and </td>
<td> `&#038;` </td>
</tr>
<tr>
<td> or </td>
<td> `|` </td>
</tr>
<tr>
<td> equals </td>
<td> `==` </td>
</tr>
<tr>
<td> is_greater_than </td>
<td> `>` </td>
</tr>
<tr>
<td> is_weakly_greater_than </td>
<td> `>=` </td>
</tr>
<tr>
<td> is_less_than </td>
<td> `<` </td>
</tr>
<tr>
<td> is_weakly_less_than </td>
<td> `<=` </td>
</tr>
<tr>
<td> not (`n&#8217;est pas`) </td>
<td> `!` </td>
</tr>
<tr>
<td> set_colnames </td>
<td> `colnames<-` </td>
</tr>
<tr>
<td> set_rownames </td>
<td> `rownames<-` </td>
</tr>
<tr>
<td> set_names </td>
<td> `names<-` </td>
</tr>
</table>

# 3. 雜七雜八
## 3.1. 參考資料
- smbache/magrittr · GitHub (https://github.com/smbache/magrittr)
- magrittr 1.5 | RStudio Blog (http://blog.rstudio.org/2014/12/01/magrittr-1-5/)
- 

## 3.2. 該知道的套件 
- R package `dplyr` for data ETL [ref](https://github.com/hadley/dplyr)
- R package `ggvis` for data Viz [ref](https://github.com/rstudio/ggvis)


    
