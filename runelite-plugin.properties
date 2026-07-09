package com.osrsnamewatcher;

import com.google.inject.Provides;
import java.io.IOException;
import java.nio.charset.StandardCharsets;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardOpenOption;
import java.time.Instant;
import javax.inject.Inject;
import lombok.extern.slf4j.Slf4j;
import net.runelite.api.Nameable;
import net.runelite.api.events.NameableNameChanged;
import net.runelite.client.config.ConfigManager;
import net.runelite.client.eventbus.Subscribe;
import net.runelite.client.plugins.Plugin;
import net.runelite.client.plugins.PluginDescriptor;

@Slf4j
@PluginDescriptor(
	name = "OSRS Name Watcher",
	description = "Writes friend-list name changes to a local file",
	tags = {"friends", "names", "discord"}
)
public class OsrsNameWatcherPlugin extends Plugin
{
	@Inject
	private OsrsNameWatcherConfig config;

	@Provides
	OsrsNameWatcherConfig provideConfig(ConfigManager configManager)
	{
		return configManager.getConfig(OsrsNameWatcherConfig.class);
	}

	@Override
	protected void startUp()
	{
		log.info("OSRS Name Watcher started");
	}

	@Override
	protected void shutDown()
	{
		log.info("OSRS Name Watcher stopped");
	}

	@Subscribe
	public void onNameableNameChanged(NameableNameChanged event)
	{
		Nameable nameable = event.getNameable();

		if (nameable == null)
		{
			return;
		}

		String newName = nameable.getName();
		String oldName = nameable.getPrevName();

		if (oldName == null || newName == null)
		{
			return;
		}

		if (oldName.equalsIgnoreCase(newName))
		{
			return;
		}

		if (!matchesWatchList(oldName) && !matchesWatchList(newName))
		{
			return;
		}

		writeEvent(oldName, newName);
	}

	private boolean matchesWatchList(String name)
	{
		String watchList = config.watchNames();

		if (watchList == null || watchList.trim().isEmpty())
		{
			return true; // Empty watchlist = log every detected name change.
		}

		String normalisedName = normalise(name);
		String[] entries = watchList.split("[,\\n\\r]+");

		for (String entry : entries)
		{
			if (normalise(entry).equals(normalisedName))
			{
				return true;
			}
		}

		return false;
	}

	private String normalise(String name)
	{
		return name == null ? "" : name.trim().toLowerCase();
	}

	private void writeEvent(String oldName, String newName)
	{
		Path outputPath = getOutputPath();

		String jsonLine = "{"
			+ "\"timestamp\":\"" + escapeJson(Instant.now().toString()) + "\","
			+ "\"old_name\":\"" + escapeJson(oldName) + "\","
			+ "\"new_name\":\"" + escapeJson(newName) + "\","
			+ "\"source\":\"runelite_nameable_changed\""
			+ "}";

		try
		{
			Files.createDirectories(outputPath.getParent());

			Files.write(
				outputPath,
				(jsonLine + System.lineSeparator()).getBytes(StandardCharsets.UTF_8),
				StandardOpenOption.CREATE,
				StandardOpenOption.APPEND
			);

			log.info("Name change written: {} -> {}", oldName, newName);
		}
		catch (IOException e)
		{
			log.error("Failed to write name change event", e);
		}
	}

	private Path getOutputPath()
	{
		String customPath = config.outputFile();

		if (customPath != null && !customPath.trim().isEmpty())
		{
			return Path.of(customPath.trim());
		}

		return Path.of(
			System.getProperty("user.home"),
			".runelite",
			"osrs-name-watcher-events.jsonl"
		);
	}

	private String escapeJson(String value)
	{
		return value
			.replace("\\", "\\\\")
			.replace("\"", "\\\"")
			.replace("\n", "\\n")
			.replace("\r", "\\r");
	}
}
