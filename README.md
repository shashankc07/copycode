# copycode
copycode


using System;
using System.Collections.Generic;
using System.IO;
using YamlDotNet.RepresentationModel;

class Program
{
    static void Main()
    {
        string yamlContent = File.ReadAllText("your_yaml_file.yaml");

        Dictionary<string, object> specificTagDictionary = DeserializeSpecificTagToDictionary(yamlContent, "your_specific_tag");

        // Now you can use the specificTagDictionary as needed
        foreach (var key in specificTagDictionary.Keys)
        {
            Console.WriteLine($"Key: {key}, Value: {specificTagDictionary[key]}");
        }
    }

    static Dictionary<string, object> DeserializeSpecificTagToDictionary(string yamlContent, string specificTag)
    {
        var input = new StringReader(yamlContent);
        var yamlStream = new YamlStream();
        yamlStream.Load(input);

        var rootNode = yamlStream.Documents[0].RootNode;

        // Assuming your specific tag is a mapping node under the root
        var specificTagNode = FindNodeWithTag(rootNode, specificTag);

        return ConvertNodeToDictionary(specificTagNode);
    }

    static YamlNode FindNodeWithTag(YamlNode node, string tag)
    {
        if (node.Tag == tag)
        {
            return node;
        }

        if (node is YamlMappingNode mappingNode)
        {
            foreach (var entry in mappingNode.Children)
            {
                var foundNode = FindNodeWithTag(entry.Value, tag);
                if (foundNode != null)
                {
                    return foundNode;
                }
            }
        }

        if (node is YamlSequenceNode sequenceNode)
        {
            foreach (var item in sequenceNode.Children)
            {
                var foundNode = FindNodeWithTag(item, tag);
                if (foundNode != null)
                {
                    return foundNode;
                }
            }
        }

        return null;
    }

    static Dictionary<string, object> ConvertNodeToDictionary(YamlNode node)
    {
        var result = new Dictionary<string, object>();

        if (node is YamlMappingNode mappingNode)
        {
            foreach (var entry in mappingNode.Children)
            {
                result.Add(entry.Key.ToString(), ConvertNodeToDictionary(entry.Value));
            }
        }
        else if (node is YamlSequenceNode sequenceNode)
        {
            for (int i = 0; i < sequenceNode.Children.Count; i++)
            {
                result.Add(i.ToString(), ConvertNodeToDictionary(sequenceNode.Children[i]));
            }
        }
        else
        {
            result.Add("Value", node.ToString());
        }

        return result;
    }
}
