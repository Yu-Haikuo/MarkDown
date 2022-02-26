# Stroop中存在的一些问题

## 架构和代码层面中的一些问题

### 文件上没有按照 MVC 分类，略有一些杂乱

### Controller 有些混乱+臃肿：/stroop/Test/Stroop/StroopBaseController.swift

- 所有的东西都被放进单个 Controller 导致文件体积过大
- 没有采用 OOP 进行抽象封装一些较低级别的数据和函数
- 部分函数逻辑有些过于复杂（10层嵌套）
- 有些缺乏注释

## 一些 UI 方面存在的问题

<img src="../_resources/Simulator Screen Shot - iPad Pro (12.9-inch) (5th -1.png" alt="Simulator Screen Shot - iPad Pro (12.9-inch) (5th generation) - 2022-02-19 at 17.43.57.png" width="450" height="600" class="jop-noMdConv"><img src="../_resources/Simulator Screen Shot - iPad mini (6th generation).png" alt="Simulator Screen Shot - iPad mini (6th generation) - 2022-02-26 at 00.49.36.png" width="394" height="600">

图左：iPad Pro (12.9-inch) 5th generation                                          图右：iPad mini (6th generation) 

### Home View 部分 UI 过小未能占满屏幕/过大导致超出屏幕范围

代码定位（其中之一）：

```Swift
func initTestCollectionView() {
        let layout = UICollectionViewFlowLayout()
        layout.sectionInset = UIEdgeInsets.init(top: 0, left: 10, bottom: 0, right: 10)
        layout.itemSize = CGSize.init(width: 255, height: 110) // 这里采用了固定的 item size
        layout.scrollDirection = .horizontal
        testCollectionView = UICollectionView.init(frame: CGRect.init(x: 0, y: -1, width: UIScreen.main.bounds.size.width, height: 132), collectionViewLayout: layout)
        testCollectionView!.tag = 1
        testCollectionView!.delegate = self
        testCollectionView!.dataSource = self
        testCollectionView!.backgroundColor =  hexStringToUIColor(hex: "#F5F5F5")
        testCollectionView!.register(TestCell.self, forCellWithReuseIdentifier: "TestCell")
        testCollectionView!.register(UINib.init(nibName: "TestCell", bundle: nil), forCellWithReuseIdentifier: "TestCell")
        testCollectionView!.allowsSelection = true
        testCollectionView!.isUserInteractionEnabled = true
        testCollectionView!.alwaysBounceHorizontal = true
    }
```

原因分析：设计 UI 大小以及相互之间距离的时候使用固定 size，未考虑不同型号 iPad 屏幕大小不同

改进：可根据 UIScreen.main.bounds.size.width 的大小来调整组件以及他们的大小

### Home View 顶部 Banner 不够长

代码定位：

```Swift
func changeBackgroundImage() {
        let h = NSCalendar.current.component(.hour, from: NSDate.now)
        var period = "morning"
        if (h < 5) {
            period = "morning"
        } else if (h < 17) {
            period = "afternoon"
        } else {
            period = "evening"
        }
        
        let headerName = "img-header-\(period)"
        
        headerView.image = UIImage(named: headerName) // Can resize image before showing
        self.navigationItem.title = "Good \(period), user"
        
    }
```

原因分析：Image 没有 resize

改进：可以用 extension extend UIImage 来加入 resize / 写一个单独的 processing function

### Task 1: Stroop 手指位置错位

<img src="../_resources/Simulator Screen Shot - iPad Pro (12.9-inch) (5th -2.png" alt="Simulator Screen Shot - iPad Pro (12.9-inch) (5th generation) - 2022-02-25 at 22.34.27.png" width="450" height="600" class="jop-noMdConv">

代码定位（其中之一）：

```Swift
@objc func mistakeStepAnimate2() {
        self.view.bringSubviewToFront(self.finger)
        UIView.animate(withDuration: 1.5) {
            self.finger.isHidden = false
            self.finger.frame.origin = CGPoint(x: 650, y: 420) //这里采用了固定的距离，没有考虑 iPad 屏幕大小
            
        } completion: { (finished) in
            UIView.animate(withDuration: 0.5) {
                self.finger.frame.origin.y -= 1
                self.finger.image = UIImage(named:"finger_click")
            } completion: { (finished) in
                self.realIdx = 7
                self.collectionView.reloadData()
                self.finger.image = UIImage(named:"finger")
            }
        }
    }
```

原因分析：采用了固定的距离，没有考虑 iPad 屏幕大小

改进：移动距离可采用相对屏幕大小的比例

## 一些已知的 Bug：

### 在 Task 1 Stroop begin test 中选择 1. Color Naming (toggle on) -> Color Naming Tutorial 再返回上一个 View 后 Voice 不会停止并可被多次触发

<img src="../_resources/Simulator Screen Shot - iPad Pro (12.9-inch) (5th -3.png" alt="Simulator Screen Shot - iPad Pro (12.9-inch) (5th generation) - 2022-02-24 at 16.43.46.png" width="450" height="600" class="jop-noMdConv"> <img src="../_resources/Simulator Screen Shot - iPad Pro (12.9-inch) (5th .png" alt="Simulator Screen Shot - iPad Pro (12.9-inch) (5th generation) - 2022-02-24 at 16.54.39.png" width="450" height="600" class="jop-noMdConv">

### /stroop/Test/Stroop/StroopBaseController 中 func calcScore() 中 lowerBound > upperBound 导致 Fatal error 程序崩溃

![Screen Shot 2022-02-22 at 13.19.04.png](https://github.com/Yu-Haikuo/MarkDown-Images/raw/main/Stroop中存在的一些问题/_resources/Screen%20Shot%202022-02-22%20at%2013.19.04.png)
