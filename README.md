# Nudity-Detection-Android

## # The coding description of my research paper
https://www.researchgate.net/publication/341851946_A_Novel_Nudity_Detection_Algorithm_for_Web_and_Mobile_Application_Development
https://arxiv.org/ftp/arxiv/papers/2006/2006.01780.pdf

## # Kotlin
## # Android Image processing
## # Nudity



class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

// Get checkNudity from imageView

        checkNudity(imageView)
    }


    fun classifySkin(r: Int, g: Int, b: Int): Boolean {


        var rgbClassifier = ((r > 95) && (g > 40) && (b > 20)
                && (Math.abs(r - g) > 15) && (r > g) && (r > b))


        var hsv = FloatArray(3)
        var currentColor = Color.rgb(r, g, b)
        Color.colorToHSV(currentColor, hsv)
        var h = hsv[0]
        var s = hsv[1]

        var hsvClassifier = (h >= 0 && h <= 50 && s >= 0.23 && s <= 0.78)


        return ((rgbClassifier && hsvClassifier))
    }

    fun checkNudity(imageView: ImageView) {


        var cnt = 0

        //Get image bitmap from Imageview
        var bitmapImage = (imageView.drawable as BitmapDrawable).bitmap

// Resize large image to speed up detection process

        if (bitmapImage.width >= bitmapImage.height && bitmapImage.width > 1000) {
            var th = bitmapImage.height / bitmapImage.width.toFloat()
            bitmapImage = Bitmap.createScaledBitmap(bitmapImage, (1000).toInt(), (1000 * th).toInt(), true)
        } else if (bitmapImage.height > bitmapImage.width && bitmapImage.height > 1000) {
            var th = bitmapImage.width / bitmapImage.height.toFloat()
            bitmapImage = Bitmap.createScaledBitmap(bitmapImage, (1000 * th).toInt(), (1000).toInt(), true)
        }

        var bm = bitmapImage.copy(Bitmap.Config.ARGB_8888, true)

        var t = 0

        val tempBitmap = Bitmap.createBitmap(bitmapImage.width, bitmapImage.height, Bitmap.Config.RGB_565)
        val tempCanvas = Canvas(tempBitmap)
        tempCanvas.drawBitmap(bitmapImage, 0f, 0f, null)
        val faceDetector = FaceDetector.Builder(applicationContext).setTrackingEnabled(false)
                .setLandmarkType(FaceDetector.ALL_LANDMARKS)
                .build()
        if (!faceDetector.isOperational()) {
            AlertDialog.Builder(this).setMessage("Could not set up the face detector!").show()
            // return false
        }
        val frame = Frame.Builder().setBitmap(bitmapImage).build()
        val faces = faceDetector.detect(frame)

        for (i in 0 until faces.size()) {
            val thisFace = faces.valueAt(i)
            var x1 = thisFace.position.x.toInt()
            var y1 = thisFace.position.y.toInt()
            val x2 = x1 + thisFace.width.toInt()
            val y2 = y1 + thisFace.height.toInt()

            for (i1 in x1 until x2 + 1) {
                for (j1 in y1 until y2 + 1) {
                    // Grab pixel description from bitmap image
                    val pixel = bitmapImage.getPixel(i1, j1)
                    val r = Color.red(pixel)
                    val b = Color.blue(pixel)
                    val g = Color.green(pixel)
                    var a = Color.alpha(pixel)
// Detect skin pixels in faces region
                    if (classifySkin(r, g, b)) {
                        facePixel++
                    }
                }
            }
        }
        bm = BitmapDrawable(resources, tempBitmap).bitmap

        for (x in 0 until bitmapImage.width) {

            for (y in 0 until bitmapImage.height) {

                val pixel = bitmapImage.getPixel(x, y)
                val r = Color.red(pixel)
                val b = Color.blue(pixel)
                val g = Color.green(pixel)
                var a = Color.alpha(pixel)
                // Detect skin pixels in overall image
                if (classifySkin(r, g, b)) {
                    skinPixel++
                    bm.setPixel(x, y, Color.rgb((t + 101) % 255, (10 + t) % 255, (110 + t) % 255))
                }

            }

        }
        imageView.setImageBitmap(bm)
        println("total skin pixel in face $facePixel")
        println("total skin pixel in overall image $skinPixel")
    }

    var facePixel = 0
    var skinPixel = 0

}
