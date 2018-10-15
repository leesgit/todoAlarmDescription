# des

# 출시 어플

- ### 설명

  - Kotlin, room livedata, MVVM 등을 이용한 일정관리 어플입니다.


- ### 핵심 코드 설명

```
- 메인 Activity

binding!!.viewmodel!!.changeState.observe(this, Observer {
            if (!it!!) {
                visibleMonth()
                binding!!.viewmodel!!.weekVisibleState.postValue(false)
            } else {
	   visibleWeek()
                binding!!.viewmodel!!.weekVisibleState.postValue(true)   
            }
        })
//changeState의 상태에 따라 엑티비티에 보여줄 프래그먼트 표시(월간, 주간)


```
- 월간 fragment

binding!!.viewmodel!!.zoomState.observe(this, android.arch.lifecycle.Observer {
            if (it!!) {
                val editor = pref.edit()
                editor.putBoolean("zoom", true)
                editor.commit()
                ViewModelProviders.of(activity!!).get(MainViewModel::class.java).zoomState.postValue(true)
            } else {
                val editor = pref.edit()
                editor.putBoolean("zoom", false)
                editor.commit()
                ViewModelProviders.of(activity!!).get(MainViewModel::class.java).zoomState.postValue(false)

            }
        })
//사용자의 요청에 따라 메인 뷰모델의 changeState의 값을 바꿔주고 
//이를 sharedPreference에 자장해 다음에 어플이 시작될때도 같은 상태를 유지합니다.
```

```
- 월간 fragment의 viewpager 아이템 frament

binding!!.linear1.getViewTreeObserver().addOnGlobalLayoutListener(object : ViewTreeObserver.OnGlobalLayoutListener {
            override fun onGlobalLayout() {
                binding!!.linear1.getViewTreeObserver().removeOnGlobalLayoutListener(this)
                width = binding!!.linear1.width.toFloat()

                linearHeight = binding!!.linear1.height
                sunHeight = binding!!.tvSunday.height

                val px = binding!!.tvSunday.getTextSize()
                textSize = px / resources.displayMetrics.scaledDensity
                height = ((linearHeight - sunHeight) / 4.toFloat()) - 2.toFloat()

                binding!!.viewmodel!!.start.postValue(true)
            }
        })
//여기서 작성할 텍스트의 크기와 높이를 계산합니다.
//하지만 이 경우 thread간의 순서가 필요함으로 livedate start를 이용합니다.
//(height구하기 이전에 setCalendar(), setCalendarZoomIn()을 호출 할 경우를 막기 위해)

 ViewModelProviders.of(activity!!).get(MainViewModel::class.java).zoomState.observe(this, android.arch.lifecycle.Observer {
            binding!!.viewmodel!!.start.observe(this, android.arch.lifecycle.Observer {
                if (it == true) {
                    if (ViewModelProviders.of(activity!!).get(MainViewModel::class.java).zoomState.value!!) {
                        val size = binding!!.viewmodel!!.dynamicMonthDayItem!!.size
                        val size2 = binding!!.viewmodel!!.dynamicNumItem!!.size
                        if (size > 0) {
                            for (i in 0..size - 1) {
                                binding!!.viewmodel!!.dynamicMonthDayItem!!.get(i).linearLayout!!.removeView(binding!!.viewmodel!!.dynamicMonthDayItem!!.get(i).textView)
                            }
                            binding!!.viewmodel!!.dynamicMonthDayItem!!.clear()
                        }
                        if (size2 > 0) {
                            for (i in 0..size2 - 1) {
                                binding!!.mainLayout.removeView(binding!!.viewmodel!!.dynamicNumItem!!.get(i))
                            }
                            binding!!.viewmodel!!.dynamicNumItem!!.clear()
                        }
                        setCalendarZoomIn()
                    } else {
                        val size = binding!!.viewmodel!!.zoomOutMonthDayItem!!.size
                        val size2 = binding!!.viewmodel!!.dynamicNumItem!!.size
                        if (size > 0) {
                            for (i in 0..size - 1) {
                                binding!!.viewmodel!!.zoomOutMonthDayItem!!.get(i).linearLayout!!.removeView(binding!!.viewmodel!!.zoomOutMonthDayItem!!.get(i).textView)
                            }
                            binding!!.viewmodel!!.zoomOutMonthDayItem!!.clear()
                        }

                        if (size2 > 0) {
                            for (i in 0..size2 - 1) {
                                binding!!.mainLayout.removeView(binding!!.viewmodel!!.dynamicNumItem!!.get(i))
                            }
                            binding!!.viewmodel!!.dynamicNumItem!!.clear()
                        }
                        setCalendar()
                        getSchedule()
                    }
                }
            })
        })
//사용자의 요청에 따라 바뀐 메인 뷰모델의 changeState의 값을 토대로
//zoom 상태에 따라 화면을 구성 (livedate start를 통해 height과 text크기가 구해졌는지 기다린 후)
//이전에 동적으로 생성 되었던 뷰들이 있는지 체크 후 제거
//setCalendar()는 작은 글씨의 기본 화면 구성
//setCalendarZoomIn()을 통해 큰 화면 구성
//getSchedule()을 통해 일정을 불러옵니다.

binding!!.viewmodel!!.tasks!!.observe(this, android.arch.lifecycle.Observer {
            binding!!.viewmodel!!.start.observe(this, android.arch.lifecycle.Observer {
                if (it == true) {
                    if (ViewModelProviders.of(activity!!).get(MainViewModel::class.java).zoomState.value == false) {
                        getSchedule()
                    }
                }
            })
        })
//room의 쿼리 결과를 livedata로 받아 일정의 변화에 따라 일정을 표시합니다. 

```


```
- 월간 fragment의 viewpager 아이템 frament의 일간 클릭 DialogFragment

fun judgeChange(position : Int) {
        if( curMonthFormat.format(adapter?.getCalendar(position)!!) != curMonthFormat.format(ViewModelProviders.of(activity!!).get(MainViewModel::class.java).date.value)||
                curYearFormat.format(adapter?.getCalendar(position)!!) != curYearFormat.format(ViewModelProviders.of(activity!!).get(MainViewModel::class.java).date.value)) {
            ViewModelProviders.of(activity!!).get(MainViewModel::class.java).date.postValue(adapter?.getCalendar(position)!!)
        }
    }

//viewPager의 포지션의 변화에 따라 현재 시간을 동기화하고 이를 이용해 월간 주간 달력들의 날짜등을 바꿉니다.(월간 주간에서도 마찬가지)

```

```
 - DialogFragment의 viewpager 아이템 fragment

if(adapter!!.itemCount>0) {
                    val rvItem = getRecyclerViewScreenshot(binding!!.rvMonthDay)
                    val day = getBitmapFromView(binding!!.constDayInfo, binding!!.constDayInfo.height + rvItem.height, binding!!.constDayInfo.width)
                    val dayInfo = overlay(day, rvItem, binding!!.constDayInfo.height)
                    dayInfo.compress(Bitmap.CompressFormat.JPEG, 100, stream);
                } else {
                    val day = getBitmapFromView(binding!!.constDayInfo, binding!!.constDayInfo.height + binding!!.rvMonthDay.height, binding!!.constDayInfo.width)
                    day.compress(Bitmap.CompressFormat.JPEG, 100, stream);
                }
// 카카오톡 이미지 보내기 버튼이 눌렀을 경우 현재 recyclerview의 아이템의 개수 체크 후 
// recyclerview의 화면 전체와 recyclerview위의 날짜표시 화면을 겹쳐서 보냅니다. (item이 0개인 경우 그냥 제목 화면만)
// (제목의 bitmap 제목의 크기와 recycler item전체의 길이를 더한 길이로 생성후 제목 밑으로 recycler를 덮어씁니다.)
```

```
 - 주간 fragment
ViewModelProviders.of(activity!!).get(MainViewModel::class.java).date.observe(this, Observer {
            binding!!.tvMonth.setText(curYearFormat.format(it))
            currentYear = curYearFormat.format(ViewModelProviders.of(activity!!).get(MainViewModel::class.java).date.value!!)
        })

// 동기화된 일정에 따라 달력의 년도 표시 합니다.
```

```
 - 주간 fragment의 viewpager 아이템 fragment

// 여기서 화면이 옆으로 회전하였을 경우 3개의 getViewTreeObserver보다 livedata의 observe가 먼저 호출되어 문제가 되었습니다.
// rx를 통해 각각의 getViewTreeObserver에 checkData?.onNext(judge)를 주어 3개의 getViewTreeObserver thread가 모두 끝날때 
// room의 쿼리의 결과 livedata observe안에  rx코드를 작성 했지만..
// 출시 코드에는 옆으로 회전을 막아 이를 사용 안했습니다.

binding!!.timeConstraintLayout.setOnTouchListener { v, event ->
            if (event.getAction() == MotionEvent.ACTION_DOWN) {
                judge = false
                if (event.x > blockW!!) {
                    for (i in 1..7) {
                        if (blockW!! * i < event.x && blockW!! * i + width!! > event.x) {
                            for (j in 0..23) {
                                if (blockH!! * j < event.y && blockH!! * j + height!! > event.y) {
                                    date = i
                                    time = j
                                    judge = true
                                }
                            }
                        }
                    }
                }
            }
            false
        }

// 일정이 없는 경우 클릭 하였을 때 위치를 계산하여 날짜 정보와 시간 정보를 선택 Activity에 보내 주기 위한 코드 입니다.
// 각각의 블록의 크기와 높이를 통해 계산 합니다.
// ex) 첫번째 줄의 2번째 아이템을 선택했을 경우 (화요일 오전 1시~1시반)

ViewModelProviders.of(activity!!).get(MainViewModel::class.java).weekSend.observe(this, Observer {
            if (it!!) {
		...
		...
	}

//주간 fragment에서 send버튼이 눌렸을 경우 사진을 찍어 보냅니다.
```

```
 - 선택 Activity 

binding!!.viewmodel!!.notiState.observe(this, Observer {
            if (it!!) {
                val size = binding!!.viewmodel!!.notiLongTimes!!.size
                for (i in 0 until size) {
                    val task = dbInsatance!!.getTaskByNotiTime(binding!!.viewmodel!!.notiLongTimes!!.get(i))
                    val aIntent = Intent(this, NotiReceiver::class.java)
                    aIntent.putExtra("title", task.title!!)
                    val preIntent = PendingIntent.getBroadcast(this, task.idx!!, aIntent, PendingIntent.FLAG_UPDATE_CURRENT)
                    alarm!!.set(AlarmManager.RTC_WAKEUP, task.notiTime!!, preIntent)
                }
                binding!!.viewmodel!!.notiState.postValue(false)
                finish()
            }
        })
// 알림 일정을 선택하면 그 시간에 따라 알람 매니저를 설정합니다.
// idx를 토대로 알람을 설정, 해지 하기 때문에 알람 캔슬의 오류가 없습니다.
// 여기서 idx를 가져오기 위해 db에 저장 후 검색을 하게 되는데.. 
// idx @PrimaryKey(autoGenerate = true)로 해놓아
// db저장 전에 idx값을 가져올수 없어 부득이 하게 db검색을 하게 되었습니다.
// (물론 db생성시 idx를 직접 할당 하면 db검색을 안해도 됩니다.) 
```


