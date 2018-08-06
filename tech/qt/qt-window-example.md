+++
categories = ["general"]
date = "2015-08-26T11:37:47-06:00"
tags = ["document"]
title = "qt window example"

+++

Resource files are compiled into the application's executable, so they can't get lost. When we refer to resources, we use the path prefix :/ (colon slash),

	locationLabel = new QLabel(" W999 ");
    locationLabel->setAlignment(Qt::AlignHCenter);
    locationLabel->setMinimumSize(locationLabel->sizeHint());

先设置最长的内容，然后根据内容，label会算出一个sizeHint，把这个Hint设为最小尺寸，label就不会遮挡内容了。
