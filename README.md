# Description 
전기차저?(Charger)는 전기차의 수요가 늘어나는 현재, 충전소에 대한 서비스가 제대로 확립 되어있지 않았습니다. 전치가 충전소를 찾는 사람들을 위한 앱으로 
지도맵을 사용하여 충전소의 위치를 보여주고 내 차정보 등록, 현재 위치, 충전소 검색, 충전소 즐겨찾기 등록의 기능들을 제공함으로써 
전기차 사용자들의 불편함을 해소시켜주는 앱입니다.
1. 내 차정보 등록
2. 구글맵 현재 위치
3. 충전소 리스트, 검색
4. 즐겨찾기 리스트
5. 내 차정보 수정

5가지 기능을 이용하였습니다.

## Environment
1. 프로그래밍 개발 환경
- PC 환경 : Windows 10
- Mobile 환경 : Samsung, LG 스마트폰


2. 프로그래밍 개발 도구 및 언어
- 개발 도구 : Android Studio
- Kotlin	

3. DB 
- SQLite

## 기능 흐름도
![image](https://user-images.githubusercontent.com/100817617/178384205-97202544-07fe-453c-bb6f-13e3413fc43d.png)
## 각 메뉴의 중요 기능 살펴보기
1. 내 차정보 등록

Spinner  기능을 이용하여 자기의 충전기 타입을 선택하였습니다.
핸드폰 앨범의 사진을 등록할 수 있게 구현 하였습니다. 
```
 val items = arrayOf("DC차데모","AC완속","DC차데모+AC3상","DC콤보","DC차데모+DC콤보","DC차데모+AC3상+DC콤보","AC3상")
        val myAdapter = ArrayAdapter(this, android.R.layout.simple_spinner_dropdown_item,items)
        binding.spinner.adapter= myAdapter
        binding.spinner.onItemSelectedListener = object  : AdapterView.OnItemSelectedListener {
            override fun onItemSelected(parent: AdapterView<*>?, view: View?, position: Int, id: Long) {
                when(position) {
                    in 0..6   ->  {
                        binding.spinner.getItemAtPosition(position)
                    }
                }
            }
            override fun onNothingSelected(p0: AdapterView<*>?) {
            }
        }
```
2.  구글맵
현재 위치를 Fragment로 불러옵니다.
```
        // 화면에서 fragment view 찾아서 거기에 지도를 맵핑하도록 동기화처리
        (supportFragmentManager.findFragmentById(R.id.mapView) as SupportMapFragment).getMapAsync(this)
        //Activity간의 이동
        binding.btnSearch.setOnClickListener {
            val intent = Intent(this@MainActivity, SearchActivity::class.java)
            startActivity(intent)
        }

        binding.FABCarInfo.setOnClickListener{
            val intent = Intent(this@MainActivity, CarInfoActivity::class.java)
            startActivity(intent)
        }

        binding.FABBookmark.setOnClickListener {
            val intent = Intent(this@MainActivity, BookmarkActivity::class.java)
            startActivity(intent)
        }
        binding.FABMain.setOnClickListener {
            toggle()
        }


```
3. 검색
 
 ```
우측 상단 옵션바를 구현하였습니다. Spinner의 기능을 활용하였으며, DB쿼리문을 이용해 아이템 선택시 해당된 충전소 위치를 다시 보여집니다.
searchView.setOnQueryTextListener(object:androidx.appcompat.widget.SearchView.OnQueryTextListener{
            //검색 완료
            override fun onQueryTextSubmit(query: String?): Boolean {
                carList?.clear()
                carList = dbHelper.selectSearchPublicData(query, selectChargeType)
                for(i in 0 until carList!!.size) {
                    Log.d("shin", "stationName: ${carList!!.get(i).stationName}, address: ${carList!!.get(i).address}")
                }
                binding.recyclerView.adapter = RecyclerViewAdapter(this@SearchActivity, carList!!, SEARCH_ACTIVITY)
                return true
            }

            override fun onQueryTextChange(query: String?): Boolean {
                if(!query.isNullOrBlank()){
                    searchString = query
                    carList?.clear()
                    carList = dbHelper.selectSearchPublicData(query, selectChargeType)
                    binding.recyclerView.adapter = RecyclerViewAdapter(this@SearchActivity, carList!!, SEARCH_ACTIVITY)
                }else{
                    carList?.clear()
                    carList = dbHelper.selectAllPublicData()
                    binding.recyclerView.adapter = RecyclerViewAdapter(this@SearchActivity, carList!!, SEARCH_ACTIVITY)
                }

                return true
            }
        })

```
4. 즐겨찾기
SearchActivity에서 즐겨찾기 등록한것들을 따로 리스트를 볼 수 있는 BookMarkActivity를 구현하였습니다. 
등록된 즐겨찾기 취소시 notifyItemChanged를 이용하여 리스트에 사라집니다. 

```
class RecyclerViewAdapter(val context: Context, var publicDatalist: MutableList<PublicData>?,val selectActivity: String):
private val BOOKMARK_ACTIVITY = "bookmark"

binding.ivEmptyStar.setOnClickListener {
            if(selectedPublicData?.bookmark == 1) {
                selectedPublicData.bookmark = 0
                dbHelper.updateBookmark(selectedPublicData)
                if(selectActivity == SEARCH_ACTIVITY){
                    notifyItemChanged(position)
                    Toast.makeText(context, "즐겨찾기 취소", Toast.LENGTH_SHORT).show()
                }else if(selectActivity == BOOKMARK_ACTIVITY){
                    publicDatalist?.clear()
                    Toast.makeText(context, "즐겨찾기 취소", Toast.LENGTH_SHORT).show()
                    publicDatalist = dbHelper.selectBookmarkPublicData()
                    notifyDataSetChanged()
                }
            } else {
                selectedPublicData?.bookmark = 1
                if(dbHelper.updateBookmark(selectedPublicData!!)) {
                    Toast.makeText(context, "즐겨찾기 성공", Toast.LENGTH_SHORT).show()
                }
                notifyItemChanged(position)
            }

```
!![image](https://user-images.githubusercontent.com/100817617/178387867-ab0108db-df5f-458d-8200-469db4a2c0ec.png)

