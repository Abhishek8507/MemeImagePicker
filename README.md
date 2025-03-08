@Composable
    fun MemeGeneratorScreen() {
        var imageUri by remember { mutableStateOf<Uri?>(null) }
        var memeText by remember { mutableStateOf("Your Meme Text") }
        var textOffset by remember { mutableStateOf(Offset(50f, 50f)) }
        val context = LocalContext.current

        val imagePicker = rememberLauncherForActivityResult(ActivityResultContracts.GetContent()) { uri: Uri? ->
            imageUri = uri
        }

        Column(
            modifier = Modifier.fillMaxSize().background(Color.White),
            horizontalAlignment = Alignment.CenterHorizontally
        ) {
            Button(onClick = { imagePicker.launch("image/*") }) {
                Text("Pick Meme Image")
            }

            Spacer(modifier = Modifier.height(10.dp))

            Box(modifier = Modifier.fillMaxSize(), contentAlignment = Alignment.TopCenter) {
                imageUri?.let {
                    Image(
                        painter = rememberAsyncImagePainter(it),
                        contentDescription = "Meme Image DOGE",
                        modifier = Modifier.fillMaxSize()
                    )

                    Canvas(modifier = Modifier
                        .fillMaxSize()
                        .pointerInput(Unit) {
                            detectTransformGestures { _, pan, _, _ ->
                                textOffset = Offset(textOffset.x + pan.x, textOffset.y + pan.y)
                            }
                        }
                    ) {
                        drawContext.canvas.nativeCanvas.drawText(
                            memeText,
                            textOffset.x,
                            textOffset.y,
                            android.graphics.Paint().apply {
                                color = android.graphics.Color.WHITE
                                textSize = 50f
                            }
                        )
                    }
                }
            }

            Button(onClick = { saveMeme(context) }) {
                Text("Save Meme ")
            }
        }
    }

fun Image(painter: Unit, contentDescription: String, modifier: Modifier) {

}

fun rememberAsyncImagePainter(it: Uri): Unit = Unit

fun saveMeme(context: Context) {
        val bitmap = Bitmap.createBitmap(1080, 1920, Bitmap.Config.ARGB_8888)
        val canvas = Canvas(bitmap)
        canvas.drawColor(android.graphics.Color.WHITE)

        val resolver = context.contentResolver
        val values = ContentValues().apply {
            put(MediaStore.Images.Media.DISPLAY_NAME, "meme_${System.currentTimeMillis()}.png")
            put(MediaStore.Images.Media.MIME_TYPE, "image/png")
            put(MediaStore.Images.Media.RELATIVE_PATH, "Pictures/Memes")
        }

        val uri = resolver.insert(MediaStore.Images.Media.EXTERNAL_CONTENT_URI, values)
        uri?.let {
            resolver.openOutputStream(it)?.use { stream ->
                bitmap.compress(Bitmap.CompressFormat.PNG, 100, stream)
            }
            Toast.makeText(context, "Meme saved!", Toast.LENGTH_SHORT).show()
        }
    }
