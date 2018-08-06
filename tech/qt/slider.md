+++
categories = ["general"]
date = "2015-08-15T11:31:52-06:00"
tags = ["document"]
title = "slider"

+++

# Qt Slider in Brief

## [QAbstractSlider Class](http://doc.qt.io/qt-5/qabstractslider.html)

|property                     |note |
|-----------------------------|-----------------------------|
|invertedAppearance : bool | default is false. true means slider at the right end of the bar when value is 0|
|invertedControls : bool |  default is false. true means slider moves to left side if value increases|
|maximum : int | |
|minimum : int | |
|orientation : Qt::Orientation | |
|pageStep : int | |
|singleStep : int | |
|sliderDown : bool | |
|sliderPosition : int | maybe be different with value if tracking is false |
|tracking : bool | true : value changes when position changes; false : value changes when slider is released |
|value : int | |

### methods

`void QAbstractSlider::actionTriggered(int action)`

This signal is emitted when the slider action action is triggered.

- QAbstractSlider::SliderNoAction
- QAbstractSlider::SliderSingleStepAdd
- QAbstractSlider::SliderSingleStepSub
- QAbstractSlider::SliderPageStepAdd
- QAbstractSlider::SliderPageStepSub
- QAbstractSlider::SliderToMinimum
- QAbstractSlider::SliderToMaximum
- QAbstractSlider::SliderMove

When the signal is emitted, the sliderPosition has been adjusted according to the action, but the value has not yet been propagated (meaning the valueChanged() signal was not yet emitted), and the visual display has not been updated. In slots connected to this signal you can thus safely adjust any action by calling setSliderPosition() yourself, based on both the action and the slider's value.

`void QAbstractSlider::triggerAction(SliderAction action) `

Triggers a slider action

## [QSlider Class](http://doc.qt.io/qt-5/qslider.html)

### Properties

|property                     |note |
|-----------------------------|-----|
|tickInterval : int | 刻度的间隔单位|
|tickPosition : TickPosition | 刻度的位置 |
