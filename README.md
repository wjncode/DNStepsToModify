微信,QQ中都有运动数据展示,在好友中的排名等,这里所提供的是通过修改健康中心的数据来达到刷数据的效果,让你成为数据中的运动达人,不过只是娱乐而已,而且微信只有第一次运行才会成功,其他时候都是读取手机本身的数据,而不对第三方修改的数据进行统计,QQ就没这个问题啦,当然数据修改不要超过100000步,太多了QQ也不认识了
<!--more-->

## 获取健康中心的读写权限

```swift
// 设置请求的权限,这里我们只获取读取和写入步数
let stepType: HKQuantityType? = HKObjectType.quantityType(forIdentifier: .stepCount)
let dataTypesToRead = NSSet(objects: stepType as Any)

// 权限请求
self.healthStore?.requestAuthorization(toShare: dataTypesToRead as? Set<HKSampleType>, read: dataTypesToRead as? Set<HKObjectType>, completion: { [unowned self] (success, error) in
    // 得到权限后就可以获取步数和写入了
    if success {
        self.fetchSumOfSamplesToday(for: stepType!, unit: HKUnit.count()) { (stepCount, error) in
        // 获取到步数后,在主线程中更新数字
        DispatchQueue.main.async {
        self.stepsNumberLabel.text = "\(stepCount!)"
        }
    }
} else {
    let alert = UIAlertController(title: "提示", message: "不给权限我还怎么给你瞎写步数呢,可以去设置界面打开权限", defaultActionButtonTitle: "确定", tintColor: UIColor.red)
    alert.show()
    }
})
```

## 读取步数
```swift
func fetchSumOfSamplesToday(for quantityType: HKQuantityType, unit: HKUnit, completion completionHandler: @escaping (Double?, NSError?)->()) {
    let predicate: NSPredicate? = self.predicateForSamplesToday()

    let query = HKStatisticsQuery(quantityType: quantityType, quantitySamplePredicate: predicate, options: .cumulativeSum) { query, result, error in

            var totalCalories = 0.0
            if let quantity = result?.sumQuantity() {
                let unit = HKUnit.count()
                totalCalories = quantity.doubleValue(for: unit)
            }
        completionHandler(totalCalories, error as NSError?)
    }

    self.healthStore?.execute(query)
}

func predicateForSamplesToday() -> NSPredicate {
    let calendar = Calendar.current
    let now = Date()
    let startDate: Date? = calendar.startOfDay(for: now)
    let endDate: Date? = calendar.date(byAdding: .day, value: 1, to: startDate!)
    return HKQuery.predicateForSamples(withStart: startDate, end: endDate, options: .strictStartDate)
}
```
## 写入数据
```swift
func addstep(withStepNum stepNum: Double) {
    let stepCorrelationItem: HKQuantitySample? = self.stepCorrelation(withStepNum: stepNum)
    self.healthStore?.save(stepCorrelationItem!, withCompletion: { (success, error) in
        DispatchQueue.main.async(execute: {() -> Void in
            if success {
                self.view.endEditing(true)
                self.addStepsTextField.text = ""
                self.refreshSteps()
                let alert = UIAlertController(title: "提示", message: "添加步数成功", defaultActionButtonTitle: "确定", tintColor: UIColor.red)
                alert.show()
            } else {
                let alert = UIAlertController(title: "提示", message: "添加步数失败", defaultActionButtonTitle: "确定", tintColor: UIColor.red)
                alert.show()
                return
            }
        })
    })
}

func stepCorrelation(withStepNum stepNum: Double) -> HKQuantitySample {
    let endDate = Date()
    let startDate = Date(timeInterval: -300, since: endDate)
    let stepQuantityConsumed = HKQuantity(unit: HKUnit.count(), doubleValue: stepNum)
    let stepConsumedType = HKQuantityType.quantityType(forIdentifier: .stepCount)
    let stepConsumedSample = HKQuantitySample(type: stepConsumedType!, quantity: stepQuantityConsumed, start: startDate, end: endDate, metadata: nil)
    return stepConsumedSample
}
```

[源码下载](https://github.com/wjncode/DNStepsToModify)
