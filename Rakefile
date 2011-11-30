require "bundler/setup"
require "erb"
require "uglifier"
require "sproutcore"

LICENSE = File.read("generators/license.js")

module SproutCore
  module Compiler
    class Entry
      def body
        "\n(function(exports) {\n#{@raw_body}\n})({});\n"
      end
    end
  end
end

def strip_require(file)
  result = File.read(file)
  result.gsub!(%r{^\s*require\(['"]([^'"])*['"]\);?\s*$}, "")
  result
end

def strip_sc_assert(file)
  result = File.read(file)
  result.gsub!(%r{^(\s)+sc_assert\((.*)\).*$}, "")
  result
end

def uglify(file)
  uglified = Uglifier.compile(File.read(file))
  "#{LICENSE}\n#{uglified}"
end

SproutCore::Compiler.output = "tmp/static"

def compile_utils_task
  SproutCore::Compiler.intermediate = "tmp/sproutcore-utils"
  js_tasks = SproutCore::Compiler::Preprocessors::JavaScriptTask.with_input "lib/**/*.js", "."
  SproutCore::Compiler::CombineTask.with_tasks js_tasks, "#{SproutCore::Compiler.intermediate.gsub(/tmp\//, "")}"
end

task :compile_utils_task => compile_utils_task

task :build => [:compile_utils_task]

file "dist/sproutcore-utils.js" => :build do
  puts "Generating sproutcore-utils.js"
  
  mkdir_p "dist"
  
  File.open("dist/sproutcore-utils.js", "w") do |file|
    file.puts strip_require("tmp/static/sproutcore-utils.js")
  end
end

# Minify dist/sproutcore-utils.js to dist/sproutcore-utils.min.js
file "dist/sproutcore-utils.min.js" => "dist/sproutcore-utils.js" do
  puts "Generating sproutcore-utils.min.js"
  
  File.open("dist/sproutcore-utils.prod.js", "w") do |file|
    file.puts strip_sc_assert("dist/sproutcore-utils.js")
  end
  
  File.open("dist/sproutcore-utils.min.js", "w") do |file|
    file.puts uglify("dist/sproutcore-utils.prod.js")
  end
end

task :dist => ["dist/sproutcore-utils.min.js"]

task :default => :dist