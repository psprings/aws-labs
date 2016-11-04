require 'optparse'
require 'digest'
require 'Date'
require 'aws-sdk-core'
require 'json'
require 'rbconfig'
require 'timeout'

$username = 'ec2-user'
os = RbConfig::CONFIG['host_os']
$os = if os.downcase.include?('linux')
        'linux'
      elsif os.downcase.include?('darwin')
        'darwin'
      else
        'windows'
      end
$default_ec2_os = 'LINUX'

class AwsLabs
  attr_reader :profile, :region, :aws_access_key_id, :aws_secret_access_key, :lab_number, :ec2_client
  attr_reader :profile, :region, :aws_access_key_id, :aws_secret_access_key, :lab_number, :ec2_client
  # Initialize the AwsLabs Client
  #
  # @param [Hash] options parameters for intantiating the client
  #  * +:profile+ [String] the name of the profile(required)
  #  * +:region+ [String] the region you want to run in (required)
  #
  # @example Instantiate the AwsLabs Client
  #   @client = SonarQube::Client.new(
  #     :host => "12.345.67.89",
  #     :port => 443,
  #     :api_token => '12325346sdfs453413'
  #   )
  #
  def initialize(options)
    @lab_number = options[:lab_number]
    @profile = create_profile_name(options)
    @aws_access_key_id = options[:aws_access_key_id] || get_aws_access_key_id
    @aws_secret_access_key = options[:aws_secret_access_key] || get_aws_secret_access_key
    @region = options[:region] || 'us-east-1'
    @os = options[:os] || 'LINUX'
    @ec2_client = create_ec2_client
  end

  def create_profile_name(options)
    lab_number = options[:lab_number]
    year_month = Date.today.strftime('%Y-%m')
    "aws-class-#{year_month}-lab#{lab_number}"
  end

  def aws_credentials
    Aws::SharedCredentials.new(profile_name: @profile_name).credentials
  end

  def get_aws_access_key_id
    aws_credentials.access_key_id
  end

  def get_aws_secret_access_key
    aws_credentials.access_key_id
  end

  def create_ec2_client
    Aws.config[:profile] = @profile
    Aws.config[:region] = @region
    Aws::EC2::Client.new
  end
end

class Setup < Thor
  desc 'new_lab', 'Configure your ~/.aws/credentials file with new lab creds'
  method_option :aws_access_key_id, :aliases => '-i', :desc => 'AWS Access Key ID'
  method_option :aws_secret_access_key, :aliases => '-k', :desc => 'AWS Secret Access Key'
  method_option :region, :aliases => '-r', :desc => 'AWS Region'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  def new_lab(options = options)
    aws_access_key_id = options[:aws_access_key_id]
    aws_secret_access_key = options[:aws_secret_access_key]
    region = options[:region]
    region ||= 'us-east-1'
    lab_number = options[:lab_number]
    year_month = Date.today.strftime('%Y-%m')
    profile_name = "aws-class-#{year_month}-lab#{lab_number}"
    path_to_creds = File.join(ENV['HOME'], '.aws', 'credentials')
    cred_text = File.open(path_to_creds).read
    hasnt_changed = false
    if cred_text =~ /\[#{profile_name}\]/
      new_content = cred_text
      hasnt_changed = (cred_text.scan(/(\[#{profile_name}\]\naws_access_key_id = )(.*)/).first[1] == aws_access_key_id) && (cred_text.scan(/(\[#{profile_name}\]\n.*\naws_secret_access_key = )(.*)/).first[1] == aws_secret_access_key)
      new_content = new_content.gsub(/(\[#{profile_name}\]\naws_access_key_id = )(.*)/, '\1' + aws_access_key_id)
      new_content = new_content.gsub(/(\[#{profile_name}\]\n.*\naws_secret_access_key = )(.*)/, '\1' + aws_secret_access_key)
      new_content = new_content.gsub(/(\[#{profile_name}\]\n.*\nregion = )(.*)/, '\1' + region) if region
    else
      new_content = cred_text + "\n[#{profile_name}]"
      new_content = new_content + "\naws_access_key_id = #{aws_access_key_id}"
      new_content = new_content + "\naws_secret_access_key = #{aws_secret_access_key}"
      new_content = new_content + "\nregion = #{region}"
    end
    File.open(path_to_creds, 'w') { |file| file.puts new_content }
    return hasnt_changed
  end
  desc 'configure_lab_key', 'Configure key for the new lab'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  def configure_lab_key(options = options)
    lab_number = options[:lab_number]
    latest_downloaded_key = Dir["#{ENV['HOME']}/Downloads/qwikLABS*.pem"].sort_by { |file_name| File.stat(file_name).mtime }.last
    latest_download_sha = Digest::SHA2.file(latest_downloaded_key).hexdigest
    latest_local_lab_keys = Dir["#{ENV['HOME']}/.ssh/aws-lab#{lab_number}.pem"].sort_by { |file_name| File.stat(file_name) }
    latest_local_sha = nil
    unless latest_local_lab_keys.empty?
      latest_local_lab_key = latest_local_lab_keys.last
      latest_local_sha = Digest::SHA2.file(latest_local_lab_key).hexdigest
    end
    if latest_download_sha != latest_local_sha
      puts "You have a new private key for Lab #{lab_number}. Updating..."
      lab_key_name = File.join(ENV['HOME'], '.ssh', "aws-lab#{lab_number}.pem")
      File.delete(lab_key_name) if File.exist?(lab_key_name)
      File.open(latest_downloaded_key, 'rb') do |input|
      File.open(lab_key_name, 'wb') do |output|
          while buff = input.read(4096)
            output.write(buff)
          end
        end
      end
      FileUtils.chmod_R 0400, lab_key_name
    end
  end
  desc 'get_instances', 'Get instances matching parameter'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  method_option :os, :aliases => '-o', :desc => 'Operating system to use'
  def get_instances(options = options)
    lab_number = options[:lab_number]
    options[:os] ||= $default_ec2_os
    os = options[:os]
    year_month = Date.today.strftime("%Y-%m")
    profile_name = "aws-class-#{year_month}-lab#{lab_number}"
    # Aws.config[:credentials] = Aws::SharedCredentials.new(profile_name: profile_name)
    Aws.config[:region] = 'us-east-1'
    Aws.config[:profile] = profile_name
    ec2 = Aws::EC2::Client.new
    ec2_filters = nil
    usefilter = "#{os.upcase} Dev Instance" if %w( LINUX WINDOWS ).include?(os)
    usefilter ||= os
    ec2_filters ||= [name: 'tag:Name', values: [usefilter]] if os
    focus_ec2 = ec2.describe_instances({filters: ec2_filters})
    public_ip = nil
    if focus_ec2.first[1].nil?
      begin
        Timeout.timeout(60) do
          while focus_ec2.first.reservations.first.nil?
            puts 'No public ip for the node yet'
            sleep 2
            focus_ec2 = ec2.describe_instances({filters: ec2_filters})
          end
        end
      rescue Timeout::Error
        puts 'Timeout waiting for EC2 instance to become active'
      end
      public_ip = focus_ec2.first.reservations.first.instances.first.public_ip_address
    else
      puts focus_ec2
    end
    puts public_ip
    return public_ip
  end
  desc 'remote_sync', 'Configure remote-sync for Atom'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  method_option :os, :aliases => '-o', :desc => 'Operating system to use'
  def remote_sync(options = options)
    lab_number = options[:lab_number]
    options[:os] ||= $default_ec2_os
    json_obj = JSON.parse("{\n  \"uploadOnSave\": true,\n  \"useAtomicWrites\": " \
    "false,\n  \"deleteLocal\": false,\n  \"hostname\": \"\",\n"" \
    \"port\": \"22\",\n  \"target\": \"\",\n" \
    "\"ignore\": [\n    \".remote-sync.json\",\n    \".git/**\"\n  ]," \
    " \"username\": \"\",\n" \
    "\"keyfile\": \"\"," \
    "\n  \"watch\": [],\n  \"transport\": \"scp\"\n}")
    json_obj['target'] = File.join('/home', $username, 'workdir')
    json_obj['username'] = $username
    lab_key_name = File.join(ENV['HOME'], '.ssh', "aws-lab#{lab_number}.pem")
    json_obj['keyfile'] = lab_key_name
    s = Setup.new
    hostname = s.get_instances(options)
    json_obj['hostname'] = hostname
    remote_sync_text = JSON.pretty_generate(json_obj)
    lab_dir = File.join(Dir.pwd, "lab#{lab_number}")
    Dir.mkdir(lab_dir) unless File.directory?(lab_dir)
    File.open(File.join(lab_dir, '.remote-sync.json'), 'w') { |f| f.write(remote_sync_text) }
  end
  desc 'easy', 'Full setup of all services'
  method_option :aws_access_key_id, :aliases => '-i', :desc => 'AWS Access Key ID'
  method_option :aws_secret_access_key, :aliases => '-k', :desc => 'AWS Secret Access Key'
  method_option :region, :aliases => '-r', :desc => 'AWS Region'
  method_option :os, :aliases => '-o', :desc => 'Operating system to use'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  def easy(options = options)
    options[:os] ||= $default_ec2_os
    s = Setup.new
    hasnt_changed = s.new_lab(options)
    unless hasnt_changed
      puts "New AWS Credentials detected."
      puts "Updated .aws/credentials"
      s.configure_lab_key(options)
      puts "Updated private key naming."
      s.remote_sync(options)
      puts "Updated remote_sync Atom configuration."
    end
  end
end

class Interact < Thor
  desc 'ssh', 'SSH Command for given OS'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  method_option :os, :aliases => '-o', :desc => 'Operating system to use'
  def ssh(options = options)
    lab_number = options[:lab_number]
    options[:os] ||= $default_ec2_os
    s = Setup.new
    username = $username
    hostname = s.get_instances(options)
    lab_key_name = File.join(ENV['HOME'], '.ssh', "aws-lab#{lab_number}.pem")
    args = []
    args += %W[ -i #{lab_key_name} ]
    args += %W[ -o UserKnownHostsFile=/dev/null ]
    args += %W[ -o StrictHostKeyChecking=no ]
    args += %W[ -o IdentitiesOnly=yes ]
    args += %W[ #{username}@#{hostname} ]
    ssh_command = 'ssh ' + args.join(' ')
    puts "You are on #{$os.upcase} so this probably won't work" if $os == 'windows'
    exec(ssh_command)
  end

  desc 'atom', 'Open project in Atom'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  def atom(options = options)
    lab_number = options[:lab_number]
    lab_dir = File.join(Dir.pwd, "lab#{lab_number}")
    command = 'atom ' + lab_dir
    puts "You are on #{$os.upcase} so this probably won't work" if $os == 'windows'
    exec(command)
  end
  desc 'get_windows_password', 'Not ready yet'
  method_option :lab_number, :aliases => '-l', :type => :numeric, :desc => 'Lab number'
  def get_windows_password(options = options)
    os = 'WINDOWS'
    aws_labs = AwsLabs.new(options)
    client = aws_labs.ec2_client
    ec2_filters = [name: 'tag:Name', values: ["#{os.upcase} Dev Instance"]] if os
    focus_ec2 = client.describe_instances({filters: ec2_filters})
    instance_id = focus_ec2.first.reservations.first.instances.first.instance_id
    password = client.get_password_data({
      dry_run: false,
      instance_id: instance_id
    })
    puts password.data.password_data
  end
end
