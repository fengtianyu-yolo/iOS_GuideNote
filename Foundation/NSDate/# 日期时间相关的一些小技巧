# 日期时间相关的一些小技巧

## 比较指定日期是否在7天内

1. 计算7天时间一共多少秒

2. 将指定日期转为时间戳

3. 将当前日期转为时间戳

4. 比较两个时间戳的差是否在7天总共秒数之内

### 示例代码

```

	// 指定时间是否在当前时间的7天内
	- (BOOL)isIn7DaysAwayToday:(NSDate *)lastDate {
		NSTimeInterval secondsPerWeek = 24 * 60 * 60 * 7; // 7天一共有多少秒
		NSTimeInterval currentTimeInterval = [[NSDate date] timeIntervalSince1970]; // 当前时间的时间戳
		NSTimeInterval lastTimeInterval = [lastDate timeIntervalSince1970]; // 上次时间的时间戳
		return (currentTimeInterval - lastTimeInterval) < secondsPerWeek;
	}

```

## 比较两个日期是否为同一天

1. 将两个日期格式化为同一格式的日期字符串

2. 比较两个字符串是否相同

### 示例代码

```
	// 指定日期是否是今天
	- (BOOL)isSameDate:(NSDate *)date {
    	NSDateFormatter *formater = [[NSDateFormatter alloc] init];
    	[formater setDateFormat:@"yyyy-MM-dd"];
    	NSString *today = [formater stringFromDate:[NSDate date]];
    	NSString *dateString = [formatter stringFromDate:date]
    	return [today isEqualToString:dateString];

	}


```

