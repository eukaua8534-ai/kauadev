from PIL import Image, ImageDraw, ImageFont
import random, math, os

W, H = 1000, 320
frames = []
random.seed(17)

# Font selection
font_paths = [
    "/usr/share/fonts/truetype/dejavu/DejaVuSans-Bold.ttf",
    "/usr/share/fonts/truetype/liberation2/LiberationSans-Bold.ttf"
]
font_path = next((p for p in font_paths if os.path.exists(p)), None)
font = ImageFont.truetype(font_path, 96) if font_path else ImageFont.load_default()
small_font = ImageFont.truetype(font_path, 18) if font_path else ImageFont.load_default()

chars = "アイウエオカキクケコサシスセソタチツテトナニヌネノ0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ{}[]<>"
columns = 70
streams = []
for i in range(columns):
    streams.append({
        "x": int(i * W / columns),
        "y": random.randint(-H, H),
        "speed": random.randint(7, 19),
        "length": random.randint(4, 12),
        "hue": random.random()
    })

for frame_i in range(24):
    im = Image.new("RGB", (W, H), (3, 5, 8))
    d = ImageDraw.Draw(im)
    # subtle dark purple glow bands
    for yy in range(H):
        strength = max(0, 1 - abs(yy - H/2)/(H/2))
        # horizontal very subtle purple haze
        col = (int(8 + 12*strength), int(4 + 3*strength), int(16 + 24*strength))
        d.line((0, yy, W, yy), fill=col)
    # matrix columns
    for s in streams:
        x = s["x"]
        y0 = (s["y"] + frame_i*s["speed"]) % (H + 180) - 90
        for k in range(s["length"]):
            y = y0 - k*17
            if 0 <= y < H:
                ch = random.choice(chars)
                if k == 0:
                    color = (205, 255, 225)
                elif s["hue"] < 0.22:
                    color = (150, 95, 255) if k < 3 else (75, 42, 150)
                else:
                    color = (0, max(70, 255 - k*18), max(35, 150-k*12))
                d.text((x, y), ch, font=small_font, fill=color)
    # dark center plate to make title readable
    d.rounded_rectangle((125, 76, 875, 244), radius=26, fill=(3, 5, 10), outline=(89, 55, 156), width=2)
    # title glow layers
    title = "KAUADEV"
    bbox = d.textbbox((0,0), title, font=font, stroke_width=0)
    tw = bbox[2]-bbox[0]
    tx = (W-tw)//2
    ty = 94
    # glow approximation
    for offset, color, sw in [(7,(42,20,92),10),(4,(83,48,160),5),(2,(0,120,70),2)]:
        d.text((tx,ty), title, font=font, fill=color, stroke_width=sw, stroke_fill=color)
    d.text((tx,ty), title, font=font, fill=(244,241,255), stroke_width=1, stroke_fill=(170,135,255))
    # accent subtitle
    subtitle = "MATRIX MODE  //  DEVELOPER IN PROGRESS"
    sb = d.textbbox((0,0), subtitle, font=small_font)
    sx = (W-(sb[2]-sb[0]))//2
    d.text((sx, 218), subtitle, font=small_font, fill=(75,255,167))
    frames.append(im)

out = "/mnt/data/kauadev_matrix.gif"
frames[0].save(out, save_all=True, append_images=frames[1:], duration=85, loop=0, optimize=True)
print(out, os.path.getsize(out))
