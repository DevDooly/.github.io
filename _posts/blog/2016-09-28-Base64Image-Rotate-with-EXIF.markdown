---
layout: post
title:  "Base64Image Rotate with EXIF"
date:   2016-09-28 00:00:00
categories: blog
---

ImageEXIFRotator Class

이미지 회전, 변환 등.

사용법 : ImageEXIFROtator.Base64RotateToString(base64String);

내용

  - web에서 이미지 업로드 중 mobile 에서 올린 이미지들은 EXIF 값이 제대로 작동하지 않아 회전이 이상하게 되는 경우가 발생. ( safari 환경에서는 잘 작동하나 chrome, explore, firefox 등 자동으로 landscape 전환 ) 이를 해결하기 위해 middle(rest java server) 단에서 해결하기로 함.

  - cli 에서 가져온 base64 의 EXIF 값을 이용하여 사진을 회전시켜 새로 return 해준다. 이를 이용하여 기존에 저장하는 방식과는 다르게 회전된 이미지를 저장시켜 어디서 불러도 이미 변환된 이미지를 가져올 수 있도록 한다.
  
public class ImageEXIFRotator {

	public static BufferedImage transformImage(BufferedImage image,
			AffineTransform transform) throws Exception {

		AffineTransformOp op = new AffineTransformOp(transform,
				AffineTransformOp.TYPE_BICUBIC);
		BufferedImage destinationImage = op.createCompatibleDestImage(image,
				(image.getType() == BufferedImage.TYPE_BYTE_GRAY)
						? image.getColorModel() : null);
		Graphics2D g = destinationImage.createGraphics();
		g.setBackground(Color.WHITE);
		g.clearRect(0, 0, destinationImage.getWidth(),
				destinationImage.getHeight());
		destinationImage = op.filter(image, destinationImage);
		return destinationImage;
	}

	public static AffineTransform getExifTransformation(ImageInformation info) {

		AffineTransform t = new AffineTransform();

		switch (info.orientation) {
		case 1:
			break;
		case 2: // Flip X
			t.scale(-1.0, 1.0);
			t.translate(-info.width, 0);
			break;
		case 3: // PI rotation
			t.translate(info.width, info.height);
			t.rotate(Math.PI);
			break;
		case 4: // Flip Y
			t.scale(1.0, -1.0);
			t.translate(0, -info.height);
			break;
		case 5: // - PI/2 and Flip X
			t.rotate(-Math.PI / 2);
			t.scale(-1.0, 1.0);
			break;
		case 6: // -PI/2 and -width
			t.translate(info.height, 0);
			t.rotate(Math.PI / 2);
			break;
		case 7: // PI/2 and Flip
			t.scale(-1.0, 1.0);
			t.translate(-info.height, 0);
			t.translate(0, info.width);
			t.rotate(3 * Math.PI / 2);
			break;
		case 8: // PI / 2
			t.translate(0, info.width);
			t.rotate(3 * Math.PI / 2);
			break;
		}
		return t;
	}

	// Inner class
	public static class ImageInformation {
		public final int orientation;
		public final int width;
		public final int height;

		public ImageInformation(int orientation, int width, int height) {
			this.orientation = orientation;
			this.width = width;
			this.height = height;
		}

		public String toString() {
			return String.format("%dx%d,%d", this.width, this.height,
					this.orientation);
		}
	}

	public static ImageInformation readImageInformation(File imageFile)
			throws IOException, MetadataException, ImageProcessingException {
		Metadata metadata = ImageMetadataReader.readMetadata(imageFile);
		Directory directory = metadata
				.getFirstDirectoryOfType(ExifIFD0Directory.class);
		JpegDirectory jpegDirectory = metadata
				.getFirstDirectoryOfType(JpegDirectory.class);

		int orientation = 1;
		try {
			orientation = directory.getInt(ExifIFD0Directory.TAG_ORIENTATION);
		} catch (MetadataException me) {
			// logger.warn("Could not get orientation");
		}
		int width = jpegDirectory.getImageWidth();
		int height = jpegDirectory.getImageHeight();

		return new ImageInformation(orientation, width, height);
	}

	public static String Base64RotateToString(String base64String)
			throws MetadataException, ImageProcessingException, Exception {

		double randomValue = Math.random();
		int intValue = (int) (randomValue * 1000 * 100) + 1;
		String imgTmpString = "imageTemp" + intValue;
		String imgTmpPath = "img/";

		File dir = new File("img");
		if (!dir.exists()) {
			dir.mkdirs();
		}

		// String(base64) -> BufferedImage
		String base64Image = base64String.split(",")[1];
		// String base64Image = memberModel.getPhoto();

		byte[] imageBytes = javax.xml.bind.DatatypeConverter
				.parseBase64Binary(base64Image);
		BufferedImage img = ImageIO.read(new ByteArrayInputStream(imageBytes));

		// bytesImage -> file image
		File oFile = new File(imgTmpPath + imgTmpString);
		oFile.createNewFile();
		FileOutputStream fos = new FileOutputStream(oFile, false);
		fos.write(imageBytes);
		fos.close();

		// bufferedImage + file image -> rotated buffedImage
		img = transformImage(img, getExifTransformation(
				ImageEXIFRotator.readImageInformation(oFile)));

		// bufferedImage -> String(base64)
		ByteArrayOutputStream os = new ByteArrayOutputStream();
		OutputStream base64 = new Base64OutputStream(os);
		ImageIO.write(img, "png", base64);
		
		oFile.delete();

		return os.toString("UTF-8");

	}

}
