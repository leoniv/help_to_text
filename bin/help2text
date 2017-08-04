#!/bin/env ruby
require 'ass_launcher'

module HelpToText
  V8UNPACK = File.expand_path('../../utils/v8unpack.exe', __FILE__)
  UNZIP = 'unzip'

  extend AssLauncher::Support::Platforms

  module Sh
    def sh(s)
      r = `#{s} 2>&1`.strip
      fail r unless $?.success?
    end
  end

  class HbkFile
    include Sh
    include AssLauncher::Support::Platforms
    include AssLauncher::Api

    HBK = '1cv8_ru.hbk'
    ZIP_FILE = 'FileStorage.data'

    attr_reader :version
    def initialize(v)
      @version = Gem::Version.new v
    end

    def thin
      @thin ||= thin_get
    end

    def thin_get
      r = thins("~> #{version}.0").last
      fail "1C V#{version} not found" unless r
      r
    end

    def src
      @src ||= platform.path File.join(File.dirname(thin.path), HBK)
    end

    def unpuck_tmp_dir
      @unpuck_tmp_dir ||= File.join(Dir.tmpdir, "hbk_file_unpuck_#{hash}", version.to_s)
    end

    def unpuck_dir
      FileUtils.mkdir_p unpuck_tmp_dir
      platform.path(unpuck_tmp_dir).realpath.to_s
    end

    def unzip_tmp_dir
      File.join(unpuck_tmp_dir, "#{ZIP_FILE}.unzip")
    end

    def unzip_dir
      FileUtils.mkdir_p unzip_tmp_dir
      platform.path(unzip_tmp_dir).realpath.to_s
    end

    def storage_zip
      File.join(unpuck_dir, ZIP_FILE)
    end

    def unpuck
      sh "#{V8UNPACK} -U '#{src.realpath}' '#{unpuck_dir}'"
    end

    def unzip
      unpuck
      sh "#{UNZIP} -Co '#{storage_zip}' -x __categories__ -x *.gif -x *.png -x *.jpg -d '#{platform.path(unzip_dir)}'"
      unzip_dir
    end

    def htm_files
      Dir.glob("#{unzip}/*")
    end

    def extract_text(destdir)
      fail "#{destdir} not a directory" unless File.directory? destdir
      htm_files.each do |html_file|
        File.write(dest_file(html_file, destdir), to_text(html_file))
      end
    end

    require 'nokogiri'
    def to_text(html_file)
      normalize(Nokogiri::HTML(html(html_file)).text)
    end

    def normalize(text)
      text.gsub!("\r\n", "\n")
      text.gsub!("\r", "\n")
      text.gsub!(/\n+/, "\n")
      text.gsub!(/\u{a0}/, ' ')
      text.gsub!(/[\.,;:]/, ' ')
      text.gsub!(/\u{2013}/, '-')

      r = ''

      text.each_line do |s|
        s.gsub!(/^\s+/, '')
        s.gsub!(/\s+$/, '')
        s.gsub!(/\t/, ' ')
        s.gsub!(/(\s)+/, '\1')
        r << "#{s}\n" unless s.strip.length.zero?
      end

      r.gsub("\n", "\n\n").strip
    end

    def html(html_file)
      File.read(html_file, encoding: 'bom|utf-8')
    end

    def dest_file(src, destdir)
      File.join(destdir, File.basename(src))
    end
  end

  def windows?
    platform.windows? || platform.cygwin?
  end

  def run(v, destdir)
    fail 'It work in Windows only!' unless windows?
    HbkFile.new(v).extract_text(destdir)
  end

  def usage
    $stderr.puts "Dumper 1C help file `#{HbkFile::HBK}' to plain text"
    $stderr.puts 'USAGE:'
    $stderr.puts '  help2text 1C_VERSION DEST_DIR'
    1
  end

  extend self
end

exit HelpToText.usage if ARGV.size != 2

HelpToText.run ARGV[0], ARGV[1]
