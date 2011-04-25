#!/usr/bin/env php
<?php

class iCheck
{
	protected static $SUMSFILENAME = 'md5sums';

	protected $baseDir = '';
	protected $fileList = array();
	protected $sumFiles = array();

	public function __construct($baseDir)
	{
		$this->baseDir = $baseDir;
		$this->fileList = $this->findFiles($baseDir, $baseDir);
	}

	protected function readSumsFile(&$result, $filename)
	{
		foreach (file($filename) as $l) {
			if (empty($l)) {
				continue;
			}

			list($hash, $file) = explode(' ', $l, 2);

			$file = trim($file);
			if (strpos($file, './') === 0) {
				$file = substr($file, 2);
			}

			$f = dirname($filename) .'/'. $file;
			$f = substr($f, strlen($this->baseDir));

			$result[$f] = $hash;
		}

		ksort($result);
	}
	
	public function check()
	{
		if (empty($this->sumFiles)) {
			echo "no sum-files found!\n";
			return false;
		}

		$filelist = array();

		foreach ($this->sumFiles as $f) {
			$this->readSumsFile($filelist, $f);
		}

		$err = 0;

		foreach ($this->fileList as $file) {
			if ($file == self::$SUMSFILENAME) {
				continue;
			}

			if (!isset($filelist[$file])) {
				echo $file . ": missing in filelist!\n";
				$err++;
			}
		}
		
		if ($err) {
			echo "\n$err errors!\n";
			exit(1);
		}

		$err = 0;

		foreach ($this->fileList as $file) {
			echo "$file...";

			if ($file == self::$SUMSFILENAME) {
				echo "skip\n";
				continue;
			}

			$hash = md5_file($file);

			if (isset($filelist[$file])) {
				if ($filelist[$file] == $hash) {
					echo "ok\n";
				} else {
					echo "wrong hash!\n";
					$err++;
				}
			} else {
				echo "missing in filelist!\n";
				$err++;
			}
		}

		foreach ($filelist as $file => $hash) {
			if (!in_array($file, $this->fileList, true)) {
				echo "$file missing!\n";
				$err++;
			}
		}

		if ($err) {
			echo "\n$err errors!\n";
			exit(1);
		} else {
			echo "\nall good.\n";
		}
	}

	public function create()
	{
		if (file_exists($this->baseDir . self::$SUMSFILENAME)) {
			echo self::$SUMSFILENAME . " already exists.\n";
			exit(1);
		}

		file_put_contents($this->baseDir . self::$SUMSFILENAME, $this->dumpHashList($this->baseDir, $this->fileList));
	}

	public function dumpHashList($b, $files)
	{
		$r = '';

		foreach ($files as $file) {
			echo "$file... ";
			$hash = md5_file($b . $file);
			echo "$hash\n";
			$r .= $hash . ' ' . $file . "\n";
		}

		return $r;
	}

	public function findFiles($path, $basePath = '')
	{
		$path = rtrim($path, '/') . '/';
		$basePath = rtrim($basePath, '/') . '/';
		$result = array();
		
		if (is_dir($path) && ($d = dir($path))) {
			while (($file = $d->read()) !== false) {
				if (basename($file) == self::$SUMSFILENAME) {
					$this->sumFiles[] = $path . $file;
					continue;
				}

				if ($file != '.' && $file != '..' && strpos($file, '.') !== 0) {
					if (is_dir($path . $file)) {
						$result = array_merge($result, $this->findFiles($path . $file, $basePath));
					} else {
						$f = $path . $file;
						if ($basePath) {
							$f = substr($f, strlen($basePath));
						}
						$result[] = $f;
					}
				}
			}
		}

		return $result;
	}

}

if (empty($_SERVER['argv'][1])) {
	echo 'usage: 
  icheck create
  icheck check
';
	exit(1);
}

$o = new iCheck(getcwd() . '/');

$command = $_SERVER['argv'][1];

if ($command == 'create') {
	if (!$o->create()) {
		exit(1);
	}
}
else if ($command == 'check') {
	if (!$o->check()) {
		exit(1);
	}
}
else {
	echo "unknown command\n";
	exit(1);
}

exit(0);

