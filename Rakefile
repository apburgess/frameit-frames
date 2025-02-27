
require 'colored'
require 'mini_magick'
require 'json'

DEVICES_TO_SKIP = [
  "Apple iMac", # we don't currently support iMac
  # some super old Android devices:
  "Nokia 220",
  "Nokia 230",
  "Nokia C3-00",
  "Nokia Asha 230",
  "Samsung Galaxy Y",
  # unsupported platforms:
  "Microsoft Lumia 950",
  "Microsoft Surface Pro",
  # unsupported sides:
  "Xiaomi Mi Mix Alpha Back",
  "Xiaomi Mi Mix Alpha Side",
  "Landscape"
]

# longer colors (more than one word) must be in the beginning
DEVICE_COLORS = [
  "Space Gray",
  "Rose Gold",
  "Jet Black",
  "Matte Black",
  "Clearly White",
  "Just Black",
  "Not Pink",
  "Silver Titanium",
  "Arctic Silver",
  "Coral Blue",
  "Maple Gold",
  "Midnight Black",
  "Midnight Green",
  "Orchid Gray",
  "Burgundy Red",
  "Lilac Purple",
  "Sunrise Gold",
  "Titanium Gray",
  "Flamingo Pink",
  "Prism Black",
  "Prism Blue",
  "Prism Green",
  "Prism White",
  "Ceramic White",
  "Oh So Orange",
  "Aura Black",
  "Aura Glow",
  "Aura Pink",
  "Aura Red",
  "Aura White",
  "Aura Blue",
  "Black",
  "White",
  "Gold",
  "Silver",
  "Blue",
  "Red",
  "Yellow",
  "Green",
  "Pink",
  "Green",
  "Gray",
  "Coral",
  "Purple"
]

# The versions / renaming scheme should be updated in case FB updates iPads
IPAD_PRO_12_9_VERSION = "4th generation"
IPAD_AIR_VERSION = "2019"
IPAD_MINI_VERSION = "2019"
IPAD_RENAMING_SCHEME = {
  'iPad Pro 11' => 'iPad Pro (11-inch)',
  'iPad Pro 13' => "iPad Pro (12.9-inch) (#{IPAD_PRO_12_9_VERSION})",
  'iPad Pro' => 'iPad Pro', # don't rename the iPad Pro 1st or 2nd gen from the 'legacy' folder
  'iPad Air' => "iPad Air (#{IPAD_AIR_VERSION})",
  'iPad Mini' => "iPad Mini (#{IPAD_MINI_VERSION})",
  'iPad' => 'iPad 10.2' # must be the last one so that only 'iPad' without any version will be renamed to 10.2
}

task(:generate_device_frames) do
  MiniMagick.configure do |config|
    config.validate_on_create = false
  end

  puts("Download and extract the latest devices from Facebook")
  puts("")
  puts("  https://facebook.design/devices")
  puts("")
  confirm
  puts("Currently the script renames iPad files according to the following scheme:")
  puts(IPAD_RENAMING_SCHEME)
  puts("Update the script if they differ from the latest Facebook devices versions or naming schemes.")
  confirm
  puts("Move the content of the zip file into './raw'")
  mkdir("raw") unless File.directory?("raw")
  system("open ./raw")
  confirm

  # Let's find all the image resources we need and copy it into the correct directory
  current_time = Time.now.to_i.to_s
  output = File.join("output", current_time)
  sh("rm -rf output")
  mkdir_p(output) unless File.directory?(output)

  image_assets = []
  devices = Hash.new { |h, k| h[k] = [] }
  (Dir["raw/Phones/*/Device/*.png"] + Dir["raw/Computers/Apple*/Device/*.png"] + Dir["raw/Tablets/*/Device/*.png"] + Dir["raw/Tablets/*/Device/Device without Pencil/*.png"] + Dir["legacy/*.png"]).each do |current_raw|
    basename = File.basename(current_raw)
    next if should_skip_device?(basename)

    cp(current_raw, output)
    output_file_path = File.join(output, basename)
    sanitized_basename = sanitize_filename(basename)
    sanitized_file_path = File.join(output, sanitized_basename)
    mv(output_file_path, sanitized_file_path) unless output_file_path == sanitized_file_path
    convert_image_resource(sanitized_file_path)
    image_assets << sanitized_basename

    x, y, width = measure_slot(sanitized_file_path)
    device_name = sanitize_device_name(sanitized_basename)
    devices[device_name] << { filename: sanitized_basename, x: x, y: y, width: width }
  end

  raise "Could not find any images, make sure to copy all extracted files into ./raw" if image_assets.count < 10

  image_assets = image_assets.sort
  File.write(File.join(output, "files.json"), image_assets.to_json)
  puts("Wrote names of available files to files.json")

  File.write(File.join(output, "version.txt"), current_time)
  puts("Wrote the current time stamp to version.txt")

  # Create `offsets.json` file
  offsets = {}
  devices.each do |device, models|
    offsets[device] = { offset: "+#{models[0][:x]}+#{models[0][:y]}", width: models[0][:width] }
  end
  offsets = offsets.sort.to_h
  File.write(File.join(output, "offsets.json"), JSON.pretty_generate({ 'portrait' => offsets }))
  puts("Wrote the slot offsets to offsets.json")

  # Check if the offsets for a device are identical for all colors
  devices.each do |device, models|
    next if models.all? { |m| m[:x] == (models[0][:x]) && m[:y] == (models[0][:y]) && m[:width] == (models[0][:width]) }
    puts
    puts("The offsets for #{device} are not identical for all colors:".red)
    models.each { |m| puts("#{m[:filename]} +#{m[:x]}+#{m[:y]}, #{m[:width]}") }
    puts("You can either keep the value in `offset.json` or replace it with a more appropriate value from above.")
    puts
  end

  cp_r(output, "output/latest")

  system("open ./output")
  puts("Done writing all the things to './output', please upload both the 'latest' and the time stamp '#{current_time}' folder to GitHub".green)
  puts("To do so, clone https://github.com/fastlane/frameit-frames, make sure to check out the `gh-pages` branch".green)
  puts("and overwrite the 'latest' folder, and copy the time stamp folder into the root.".green)
  puts("Push the changes, and run `fastlane frameit update_frames` to ensure everything worked as expected".green)
end

def should_skip_device?(device_file_name)
  DEVICES_TO_SKIP.each do |device_to_skip|
    return true if device_file_name.include?(device_to_skip)
  end
  false
end

def confirm
  puts("Press any key when you're ready to continue".yellow)
  STDIN.gets
end

# Facebook's device naming is inconsistent. This method fixes the file names.
def sanitize_filename(filename)
  perform_ipad_renaming(filename
      .gsub('Grey', 'Gray') # some Apple devices have "Grey" instead of "Gray" color -> unify
      .gsub(' - Portrait', '') # iPads Pro include Portrait and Landscape - we just need Portrait; Landscape filtered via DEVICES_TO_SKIP
      .gsub(' - ', ' ') # Google Pixel device names are separated from their colors by a dash -> remove
      .gsub(' — ', ' ') # Some Apple devices have a long dash -> replace
      .gsub(' – ', ' ') # Some Apple devices have a special dash -> replace)
      .gsub('Note10', 'Note 10') # Samsung Galaxy Note 10 is missing a space in "Note10"
      .gsub('Mi Mix Alpha Front', 'Mi Mix Alpha')) # Xiaomi Mi Mix Alpha contains the words "Front", "Back" and "Side" => back and side are filtered via DEVICES_TO_SKIP, "Front" removed from the name here
end

# Facebook doesn't include versions in iPad files - this function makes sure the correct versions are added
# to the file names so that frames of older iPad versions are not accidentally overwritten by newer versions
# to ensure backward compatibility.
def perform_ipad_renaming(filename)
  if filename.include?('iPad')
    IPAD_RENAMING_SCHEME.each do |fb_name, frameit_name|
      renamed = filename.gsub!(fb_name, frameit_name)
      return renamed unless renamed.nil?
    end
  end
  filename
end

# This will remove the white space around the actual frame
def convert_image_resource(path)
  puts("Updating and trimming image file '#{path}'")
  image = MiniMagick::Image.open(path)
  image.combine_options do |co|
    co.fuzz("90%") # color dissimilarity threshold to get all shadows, see https://github.com/fastlane/fastlane/pull/14199
    co.trim("+repage") # `+repage` removes meta-data
  end
  image.write(path)
end

def measure_slot(path)
  tmp_output = Time.now.to_i.to_s + ".png"

  begin
    command_output = MiniMagick::Tool::Magick.new do |magick|
      magick << path

      # some devices (e. g. Samsung Galaxy S9) may have shadows inside their screen to show the round edges - we need
      # to remove these shadows, otherwise "connected-components" could not detect the region (would return "too many objects" error):
      magick.channel("alpha") # we work on the alpha channel
      magick.threshold("99%") # let all semitransparent pixels become fully transparent
      # rubocop:disable Lint/Void
      magick.channel.+ # reset the channel (we work on all channels again)
      # rubocop:enable Lint/Void
      magick.alpha("background") # let all transparent pixels have the same background color so that the "connected-components" detects them as a single region

      magick.negate # it makes white, black and black, white, adjusting all the colors to match
      # connected-component analysis uniquely labels connected components in an image
      magick.define("connected-components:verbose=true") # it enables a verbose output to the shell console
      magick.define("connected-components:area-threshold=100") # it eliminates small objects by merging them with their larger neighbors
      magick.connected_components("8") # it visits 8 neighbors
      magick.auto_level
      magick << tmp_output
    end
  rescue StandardError => e
    puts(e)
    puts("The functionality of slot measurements does not work with ImageMagick older than 7.x".red)
    exit!
  end

  sh("rm #{tmp_output}", verbose: false)

  # The example of the command output:
  #
  # Objects (id: bounding-box centroid area mean-color):
  #   494: 640x1136+67+239 386.5,806.5 727040 srgb(255,255,255)
  #   4: 767x1605+0+0 385.4,801.2 478199 srgb(0,0,0)
  #   522: 115x1004+0+601 12.1,1244.6 10023 srgb(255,255,255)
  #   0: 517x232+0+0 125.3,36.3 8303 srgb(255,255,255)
  #   8: 131x115+636+0 731.1,26.2 3908 srgb(255,255,255)
  #   561: 108x108+659+1497 740.4,1578.0 2954 srgb(255,255,255)
  #   500: 7x48+0+301 3.0,323.7 325 srgb(255,255,255)
  #   511: 7x42+0+454 3.0,473.7 283 srgb(255,255,255)
  #
  raw_dimensions = command_output.split("\n")[1].strip
  dimensions = /(\d+)x(\d+)\+(\d+)\+(\d+)/.match(raw_dimensions)
  x = dimensions[3]
  y = dimensions[4]
  width = dimensions[1]
  height = dimensions[2]
  return x.to_i, y.to_i, width.to_i, height.to_i
end

def sanitize_device_name(filename)
  basename = File.basename(filename, File.extname(filename))
  basename = basename.gsub("Apple", "")
  basename = basename.gsub("-", " ")

  # Directory is named "Nexus 5X" but file named "Nexus 5x"
  # Need to rename to "Nexus 5X"
  basename = basename.gsub("Nexus 5x", "Nexus 5X")

  DEVICE_COLORS.each { |color| basename.slice!(color) }
  basename.strip.to_s
end

task(default: [:generate_device_frames])
