```powershell

  param(
    [Parameter(DontShow = $true)]
    [System.String]
    $name = "Database Throughput Unit",

    [Parameter(DontShow = $true)]
    [System.String]
    $path = "S:/Data_X/diag_01/DatabaseThroughputUnit",

    [Parameter(DontShow = $true)]
    [System.Object]
    $LogFiles = @( 0x0000, 0x0003 )
  )

  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::TLS12

  Try {

    $CollectorSet = New-Object -COM Pla.DataCollectorSet

    Try {
      $CollectorSet.Query( $name, $null )
      if ( $CollectorSet.Status -eq 0 ) {
          $CollectorSet.start($true)
      }
    }
    Catch        {
      Write-Information "Creating DataCollectors for Database Throughput Unit."
      foreach ( $LogFileFormat in $LogFiles ) {
        switch ($LogFileFormat) {
          0x0000 { $LogFileName = "sql-dtu-csv"; }
          0x0003 { $LogFileName = "sql-dtu-blg"; }
        }
        $Collector                       = $CollectorSet.DataCollectors.CreateDataCollector(0);
        $Collector.Name                  = $LogFileName;
        $Collector.FileName              = "sql-dtu-log-";
        $Collector.FileNameFormat        = 0x0001;
        $Collector.FileNameFormatPattern = "yyyyMMddHHmmss";
        $Collector.SampleInterval        = 1;
        $Collector.SegmentMaxRecords     = 3600;
        $Collector.LogFileFormat         = $LogFileFormat;
        $Collector.PerformanceCounters   = @(
          "\Processor(_Total)\% Processor Time",
          "\LogicalDisk(_Total)\Disk Reads/sec",
          "\LogicalDisk(_Total)\Disk Writes/sec",
          "\SQLServer:Databases(_Total)\Log Bytes Flushed/sec"
        );
        $CollectorSet.DataCollectors.Add($Collector);
      }

      Write-Information "Creating Schedule for $name DataCollectors."
      $Schedule           = $CollectorSet.Schedules.CreateSchedule();
      $Schedule.Days      = 127;
      $Schedule.StartDate = [DateTime]$(Get-Date -Format "dd MMM yyyy 00:00:00");
      $Schedule.StartTime = [DateTime]$(Get-Date -Format "dd MMM yyyy 00:00:00");

      Write-Information "Creating the $name Data Collector Set."
      $CollectorSet.DisplayName               = $name;
      $CollectorSet.Description               = "Database Throughput Unit (DTU): DTUs provide a way to describe the relative capacity of a performance level of Basic, Standard, and Premium databases.";
      $CollectorSet.Segment                   = $true;
      $CollectorSet.Duration                  = $(3600 * 24);
      $CollectorSet.RootPath                  = $($path).Replace("/","\");

      ##  setup data collector set.
      $CollectorSet.Schedules.Add($Schedule)
      $null = $CollectorSet.Commit( $name, $null, 0x0003 );
      $CollectorSet.Query( $name, $null )

      $CollectorSet.start($true)
    }
  }
  Catch [System.Exception] {
    Write-Information "Error at line: $(($PSItem.InvocationInfo.line).Trim())"
    if ( $Error.Count > 0 ) {
      Write-Information $($Error[0].Exception.Message)
    }
    $PSCmdlet.ThrowTerminatingError($PSItem)
  }

```