# Tesseract

## 개요
- 1984~1994년도에 HP연구소에서 개발
- 광학 문자인식 엔진으로 쉽게 말해서 이미지에서 문자를 추출해주는 프로그램
- 사용해본 경험상 정확도가 그리 좋지 못하고, 다른 경험들을 찾아보면 속도또한 다른 방법들에 비해 좋지 못하다고 함

## 사용방법
1. 의존성 추가(build.gradle(:app))  
  ```
  dependencies {
    ..
    implementation 'com.rmtheis:tess-two:9.1.0'
  }
  ```
  - 추가하는 tess-two라이브러리의 경우 더이상 추가 업데이트나 유지관리를 하지 않기 때문에 해당 github에서 예시로 든 [Tesseract4Android](https://github.com/adaptech-cz/Tesseract4Android)와 같은 라이브러리로 대체하여 사용가능</br></br>

2. [Tesseract OCR github data모음](https://github.com/tesseract-ocr/tessdata)에서 필요한 언어별 학습 데이터를 다운로드</br></br>

3. (Android Studio기준)res폴더에 우클릭
  -> New -> Folder -> Assets Folder 로 assets폴더 생성</br></br>
  
4. 생성된 assets폴더 하위에 tessdata폴더 생성</br></br>

5. 2에서 다운받은 언어별 학습데이터들을 4에서 생성한 tessdata폴더 하위로 이동</br></br>

6. 이하 예시에서는
  ```
  var tessApi: TessBaseAPI
  var dataPath: String = ""
  ```
  이 두가지 객체를 전역변수로 사용하였으나 필요한 생명주기에 맞춰 선언 및 사용  </br></br>

7. assets/tessdata/에 있는 traineddata파일을 tesseract가 사용할 수 있는 폴더로 복사하는 method 구현
  ```kotlin
    private fun copyTraineddataFile(lang: String){
        try {
            val filepath = "$dataPath/tessdata/$lang.traineddata"
            val outputFile = File(filepath)
            val assetManager: AssetManager = resources.assets
            val inputStream: InputStream = assetManager.open("$lang.traineddata")

            outputFile.outputStream().use {
                    fileOutput ->  inputStream.copyTo(fileOutput)
            }
            inputStream.close()
        } catch (e: FileNotFoundException){
            e.printStackTrace()
        } catch (e: IOException) {
            e.printStackTrace()
        }
    }
  ```
  </br></br>
  
8. 언어별 학습데이터파일이 tesseract가 사용할 수 있는 경로에 존재하는지 확인하는 method 구현
  ```kotlin
    private fun checkFile(dir: File, lang: String) {
        //디렉터리가 없지만, 해당경로에 디렉터리 생성을 성공할 경우
        if(!dir.exists() && dir.mkdirs()) {
            copyTraineddataFile(lang)
        }
        //디렉터리는 존지하지만 해당 디렉터리 내에 data파일이 없는 경우
        if(dir.exists()) {
            val dataFilePath: String = "$dataPath/tessdata$lang.traineddata"
            val dataFile = File(dataFilePath)
            if(!dataFile.exists()) {
                copyTraineddataFile(lang)
            }
        }
    }
  ```
  </br></br>
  
9. Tesseract관련 초기화 method 구현
  ```kotlin
    private fun initTesseract() {
        dataPath = "$filesDir/tesseract/"
        checkFile(File("${dataPath}/tessdata"), "kor")
        checkFile(File("${dataPath}/tessdata"), "eng")
        val lang = "kor+eng"
        tessApi = TessBaseAPI()
        tessApi.init(dataPath, lang)
    }
  ```
  </br></br>

10. tesseract를 이용해 Bitmap객체에서 String으로 변환하여 반환시키는 method 구현
  ```kotlin
    private fun changeBitmapToString(bitmap: Bitmap): String {
        var ocrResult: String = ""
        tessApi.setImage(bitmap)
        ocrResult = tessApi.utF8Text

        return ocrResult
    }
  ```
  </br></br>
  
11. 10에서 구현한 변환한수 사용전 9에서 구현한 초기화 method를 실행하고, 변환할 사진이 drawable파일일 경우
  ```kotlin
  changeBitmapToString(BitmapFactory.decodeResource(getResources(), R.drawable.목표사진파일이름))
  ```
  와 같이 사용
  </br></br>
  
## 참고자료
- https://github.com/tesseract-ocr/tesseract
- https://github.com/tesseract-ocr/tessdata
- https://github.com/rmtheis/tess-two
- https://choijava.tistory.com/68
- https://brunch.co.kr/@tmapmobility/15
