## OpenNI 2 的錯誤處理 (作者： Heresy Ku )

在上一篇的 [《OpenNI 2 基本程式範例》](http://kheresy.wordpress.com/2012/12/24/openni2-basic-example/) 裡，Heresy 基本上是先整理了一下，要使用 Visual Studio 來進行 OpenNI 2 的程式開發的話，要怎樣進專案的設定，另外也用一個最簡單的例子，來說明 OpenNI 2 的程式要怎麼寫。而這一篇，則是再繼續做補充，讓程式更完整。

在上一個範例裡面，Heresy 為了版面、以及簡化程式碼的關係，是假設程式執行都沒有問題，所以把所有錯誤的檢查都拿掉了。不過實際上，程式在執行的時候，其實都是應該要考慮到各種錯誤狀況的！而 OpenNI 2 也有提供一些簡單的介面，可以用來檢查程式執行時，有沒有錯誤。

首先，和 OpenNI 1.x 的時候，OpenNI 大部分的函式，都會回傳一個代表結果的值，讓開發者可以據此判斷該函式是否已正確執行；而在 OpenNI 2，這個回傳的結果，是一個叫做 openni::Status 的列舉型別。基本的使用狀況，大致上如下：

```CPP
openni::Status eRes = openni::OpenNI::initialize();
if( eRes != openni::STATUS_OK )
{
  std::cerr << openni::OpenNI::getExtendedError() << std::endl;
  return -1;
}
```

如果函式有正確執行的話，所得到的回傳值會是 openni::STATUS_OK；反過來說，只要回傳值不是 STATUS_OK， 就代表函式執行是有問題的。

而基本上 openni::Status 已經定義的一些常見的錯誤狀況，可以用來做進一步處理的判斷。不過如果是想要得到文字性的錯誤說明的話，也可以透過 openni::OpenNI::getExtendedErropr() 這個函式，來取得更完整的錯誤說明文字。不過要注意的是，他取得的會是最後一筆錯誤資訊，如果之後又有呼叫其他函式的話，可能會影響到它的內容。（不過他是 thread-safe 的）
而如果把之前的範例，全部都加上錯誤檢查的話，則會變成類似這樣子：

```cpp
// STL Header
#include <iostream>
 
// 1. include OpenNI Header
#include "OpenNI.h"
 
// using namespace
using namespace std;
using namespace openni;
 
int main( int argc, char** argv )
{
  // 2. initialize OpenNI
  if( OpenNI::initialize() == STATUS_OK )
  {
    // 3. open a device
    Device devAnyDevice;
    if( devAnyDevice.open( ANY_DEVICE ) == STATUS_OK )
    {
      // 4. create depth stream
      VideoStream streamDepth;
      if( streamDepth.create( devAnyDevice, SENSOR_DEPTH ) == STATUS_OK )
      {
        if( streamDepth.start() == STATUS_OK )
        {
          // 5 main loop, continue read
          VideoFrameRef frameDepth;
          for( int i = 0; i < 100; ++ i )
          {
            // 5.1 get frame
            if( streamDepth.readFrame( &frameDepth ) == STATUS_OK )
            {
              // 5.2 get data array
              const DepthPixel* pDepth = (const DepthPixel*)frameDepth.getData();
 
              // 5.3 output the depth value of center point
              int idx = frameDepth.getWidth()*(frameDepth.getHeight()+1)/2;
              cout << pDepth[idx] << endl;
            }
            else
            {
              cerr << "Can not read frame\n";
              cerr << OpenNI::getExtendedError() << endl;
            }
          }
        }
        else
        {
          cerr << "Can not start depth stream\n";
          cerr << OpenNI::getExtendedError() << endl;
        }
 
        streamDepth.destroy();
      }
      else
      {
        cerr << "Can not create depth stream\n";
        cerr << OpenNI::getExtendedError() << endl;
      }
 
      devAnyDevice.close();
    }
    else
    {
      cerr << "Can not open device\n";
      cerr << OpenNI::getExtendedError() << endl;
    }
 
    // 7. shutdown
    OpenNI::shutdown();
  }
  else
  {
    cerr << "OpenNI initialize error\n";
    cerr << OpenNI::getExtendedError() << endl;
  }
 
  return 0;
}
```

這裡比較不一樣的是，Heresy 在前面有加上 using namespace openni;， 指定去使用 openni 這個 namespace，所以之後的程式，都可以把 namespace 省略掉；如此一來，程式寫起來會再簡短一點。
當然，上面的寫法也不是唯一的錯誤檢查的方法。像在官方範例「SimpleRead」裡面，採用的就是另一種程式風格的流程，有興趣的也可以看看。要採用哪種，基本上就是看人習慣了～只是另外也要提一下，理論上在出現錯誤時，main() 應該也要回傳非 0 的錯誤代碼的，Heresy 這邊沒有特別去處理這一塊就是了。

不過…既然都是 C++ 的 API 了，沒有採用 exception（參考）來做處理…Heresy 個人是覺得有點可惜啊…總覺得以各方面來說，OpenNI 的開發團隊，似乎對 C++ 不是很熟悉？雖然 OpenNI 1.x 和 OpenNI 2 都提供了 C++ 的 API，但是實際上，很多介面設計的方式，都還是用 C 的形式來做的…還是其實是有其他考量？所以甚至連陣列都是另外寫一個自己的版本（openni::Array），而沒有直接採用 STL 的版本（也沒有 iterator 可以用）…

【本文來自 Heresy's Space 的網誌，原文網址為： <http://kheresy.wordpress.com/2012/12/26/openni-error-handle/> ，由 Heresy 捐出網誌給程式人雜誌，經陳鍾誠編輯後納入雜誌】
