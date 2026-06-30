require "bundler/gem_tasks"
require 'rspec/core/rake_task'

RSpec::Core::RakeTask.new(:spec)

# Downloads a GeoLite2 database into spec/cache using MaxMind's direct
# download endpoint, which authenticates with an account ID and a license key
# via HTTP basic auth.
# See https://dev.maxmind.com/geoip/updating-databases#directly-downloading-databases
def download_maxmind_db(edition)
  dest = "spec/cache/#{edition}.mmdb"
  return if File.exist?(dest)

  account_id = ENV.fetch('MAXMIND_ACCOUNT_ID') do
    abort 'MAXMIND_ACCOUNT_ID is required to download MaxMind databases'
  end
  license_key = ENV.fetch('MAXMIND_API_KEY') do
    abort 'MAXMIND_API_KEY is required to download MaxMind databases'
  end

  url = "https://download.maxmind.com/geoip/databases/#{edition}/download?suffix=tar.gz"
  tarball = "spec/cache/#{edition}.tar.gz"

  # Pass the credentials to curl through the environment rather than the
  # command line so they never appear in the process argument list (visible via
  # `ps`) or in any echoed command, keeping the CI runner's secret masking
  # effective.
  sh({ 'MAXMIND_CREDS' => "#{account_id}:#{license_key}" },
     "curl --fail --silent --show-error --location" \
     ' --user "$MAXMIND_CREDS"' \
     " --output '#{tarball}' '#{url}'")

  # The archive contains a dated directory, e.g.
  # GeoLite2-Country_20240101/GeoLite2-Country.mmdb
  member = %x{tar tzf #{tarball}}.lines.map(&:strip).find do |entry|
    entry.end_with?("#{edition}.mmdb")
  end
  abort "#{edition}.mmdb not found in #{tarball}" unless member

  sh "tar xzf #{tarball} -C spec/cache --strip-components=1 '#{member}'"
  rm tarball
end

desc "Downloads MaxMind free DBs if required"
task :ensure_maxmind_files do
  download_maxmind_db('GeoLite2-City')
  download_maxmind_db('GeoLite2-Country')
end

desc "Downloads MaxMind free DBs if required and runs all specs"
task ensure_maxmind_files_and_spec: [:ensure_maxmind_files, :spec]

task default: :ensure_maxmind_files_and_spec
