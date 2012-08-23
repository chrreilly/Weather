#!/usr/bin/perl
use strict;
use warnings;

use LWP::Simple;

use constant ZIP_FILE => "zips.txt";

# Get location
my $defaultCity = "FRANKLIN";
my $city = prompt( "Enter a city", $defaultCity);

my $defaultState = "MA";
my $state = prompt( "Enter a state", $defaultState);

#Get zip based on location
my $zip = getZip($city, $state);

#Get weather content for location
my $site = "http://www.weather.com/weather/right-now/$zip";
my $ua = LWP::UserAgent->new();

my $response = $ua->get($site);
my @content = split /\n/, $response->decoded_content;

#Parse content to get weather information
my ( $temp, $weather, $feelslike, $winddir, $windspeed, $humidity, $dewpoint, $visibility, $pressure, $uv, $daylight, $next );
foreach ( @content ) {
    $temp = $1 if ( $_ =~ /temperature-fahrenheit">(\d+)/ );
    $weather = $1 if ( $_ =~ /weather-phrase">(.+)</ );
    $feelslike = $1 if ( $_ =~ /feels-like-temperature-fahrenheit">(\d+)/ );
    $winddir = $1 if ( $_ =~ /wx-dir-arrow wind-dir-(\w+)">/ );
    $windspeed = $1 if ( $_ =~ /wx-temp">(\d+)/ );
    $humidity = $1 if ( $_ =~ /humidity">(\d+)/ );
    $dewpoint = $1 if ( $_ =~ /dewpoint">(\d+)/ );
    $visibility = $1 if ( $_ =~ /visibility-mile">(\d+)/ );
    $pressure = $1 if ( $_ =~ /barometric-pressure-incheshg">(\d+)/ );
    $uv = $1 if ( $_ =~ /uv-index">(.+)</ );
    $daylight = $1 if ( $_ =~ /wx-value">(\d+\s+\w+\s+\d+\s+\w+)/ );
    $next = $_ if ( $_ =~ /with temperatures/ );
    if ( defined $next ) {
	$next =~ s/^\s+//;
	$next =~ s/\s+$//;
    }
}

print   "\n------------------------------------\n",
	"Current weather for $city, $state:",
	"\n------------------------------------\n",
	"Temperature: $temp F\n",
	"Feels like: $feelslike F\n",
	"Weather: $weather\n",
	"Winds: $windspeed mph in the $winddir direction\n",
	"Humidity: $humidity%\n",
	"Dewpoint: $dewpoint deg\n",
	"Visibility: $visibility miles\n",
	"Barometric pressure: $pressure inches\n",
	"UV-Index: $uv\n",
	"Remainging daylight: $daylight\n",
	"Next 6 hours: $next\n";


sub getZip {
    my ( $city, $state ) = @_;
    my $zip;

    print "Getting zipcode for $city, $state\n";
    
    open( FILE, "<", ZIP_FILE ) || die "Cannot open " . ZIP_FILE . ".\n";

    while( my $line = <FILE> ) {
	if ( $line =~ /(\d+)","$state","$city"/ ) {
	    $zip = $1;
	    print "Found zipcode: $zip\n";
	    last;
	}
    }

    close( FILE );

    if ( defined $zip ) {
	return $zip;
    } else {
	die "Could not find zipcode for $city, $state\n";
    }
}

sub prompt {
    my ( $prompt, $default ) = @_;

    print "$prompt [$default]: ";
    my $input = <STDIN>;
    chomp( $input = uc($input) );

    return $input || $default;
}
