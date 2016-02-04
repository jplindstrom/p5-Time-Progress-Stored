p5-Time-Progress-Stored
=======================

## NAME

Time::Progress::Stored - Report progress + store and retrieve the current status

## DESCRIPTION

This module helps if you have a long running process which reports
progress, but you need to actually display the progress to the user in a
different process.

Typically this is a long running web request in the web server or in a
job queue qorker, while the web browser periodically sends ajax requests
to get updated on the current progress status to show in a progress bar.

Time::Progress::Stored stores the progress report as the worker performs
its job, and retrieves it from elsewhere where the report is needed.

The report is a hashref with the following details:

    | id                | "progress-1454518121172"   |
    | max               | 262                        |
    | current           | 2                          |
    | activity          | "Importing Business Units" |
    | elapsed_seconds   | "1"                        |
    | elapsed_time      | " 0:01"                    |
    | finish_time       | "Wed Feb 3 16:50:54 2016"  |
    | percent           | 0.8                        |
    | percent_string    | " 0.8%"                    |
    | remaining_seconds | "130"                      |
    | remaining_time    | " 2:10"                    |
    | is_done           | 0                          |

## SYNOPSIS

    ### In the worker process

    my $progess_id = ...; # Probably comes from the client
    my $max = @items * 2; # Let's say two passes over @items

    my $redis = Redis->new( ... );
    my $progress = Time::Progress::Stored->new({
        max         => $max,
        progress_id => $progress_id,
        storage     => Progress::Stored::Storage::Redis->new({ redis => $redis }),
    });

    # Do the work
    for my $item (@items) {
        # Do stuff
        $progress->advance("Doing the first thing");
    }
    for my $item (@items) {
        # Do other stuff
        $progress->advance("Doing other stuff with $item->{name}");
    }

    $progress->done();


    ### HTTP endpoint, e.g. a Mojo controller for /progress/:progress_id
    sub get {
        my $self = shift;
        my $progress_id = $self->param("progress_id");

        my $storage = Time::Progress::Stored::Storage::Redis->new({
            redis => $self->redis,
        });
        my $response_json = $storage->retrieve($progress_id)
            or return $self->render(status => 404, json => {});

        return $self->render(json => { progress => $response_json });
    }

See [Time::Progress::Stored](https://metacpan.org/pod/Time::Progress::Stored) on metacpan.org

