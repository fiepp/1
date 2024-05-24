package graph.impl;

import graph.core.*;
import graph.util.DLinkedList;


/**
 * Implementation of a simple undirected graph using adjacency list.
 * <ul>
 *   <li>Edges are undirected.</li>
 *   <li>Self loop edges are not allowed.</li>
 *   <li>Parallel edges are not allowed.</li>
 * </ul>
 * The design of this class:
 * <ul>
 *   <li>Use {@code AdjVertex} to store each vertex. Each vertex has
 *   an internal linked-list to store edges which connected tos this vertex.
 *   </li>
 *   <li>Use {@code AdjEdge} to store each edge. Each edge has information of vertex position.
 *   So it's easy to lookup vertex from edge, only take O(1) time.
 *   </li>
 * </ul>
 *
 * @param <V> generic type for vertex
 * @param <E> generic type for edge
 */
public class AdjacencyListGraph<V, E> implements IGraph<V, E> {
/**
 * Internal class to store vertex.
 * <p>
 * members introduction:
 * <ul>
 *   <li>{@code value} store the element of vertex.</li>
 *   <li>{@code node} store the position of vertex in vertex list, aka {@code vertices}</li>
 *   <li>{@code affiliate} store the position of edge in edge list.</li>
 * </ul>
 */
class AdjVertex implements IVertex<V> {
  V element;
  IPosition<IVertex<V>> vertexPosition;
  IList<IPosition<IEdge<E>>> edgePosition = new DLinkedList<>();

  // constructor
  public AdjVertex(V v) {
    element = v;
  }

  // interface from IPosition
  @Override
  public V element() {
    return element;
  }

  // comparison function fo vertex
  @Override
  public boolean equals(Object o) {
    if (o != null && o.getClass() == AdjVertex.class) {
      var vertex = (AdjVertex) o;
      return vertex.element == element;
    }
    return false;
  }

  @Override
  public String toString() {
    var sb = new StringBuilder();
    sb.append("{value:");
    sb.append(element);
    sb.append(", connected edges:[");
    var it = edgePosition.iterator();
    var sep = "";
    while (it.hasNext()) {
      var edge = it.next().element();
      sb.append(sep);
      sb.append(edge.element());
      sep = ",";
    }
    sb.append("]}");
    return sb.toString();
  }
}

/**
 * Internal class to store edge.
 * <p>
 * members introduction:
 * <ul>
 *   <li>{@code element} store the element of edge.</li>
 *   <li>{@code node} store the position of edge in edge list, aka {@code edges}</li>
 *   <li>{@code srdHeadPosition} store the position of source vertex in vertex list, aka {@code vertices}</li>
 *   <li>{@code srcpos} store the position of edge in source vertex's affiliate list.</li>
 *   <li>{@code dst} store the position of destination vertex in vertex list, aka {@code vertices}</li>
 *   <li>{@code dstpos} store the position of edge in destination vertex's affiliate list.</li>
 * </ul>
 */
class AdjEdge implements IEdge<E> {
  E element;
  // position in list `edges`
  IPosition<IEdge<E>> edgeIPosition;

  // vertices connected to this edge
  IPosition<IVertex<V>> srcVertexPosition;
  IPosition<IPosition<IEdge<E>>> meInSrcListPosition;

  IPosition<IVertex<V>> dstVertexPosition;
  IPosition<IPosition<IEdge<E>>> meInDstListPosition;

  // constructor
  AdjEdge(E e, AdjVertex srcVertexPosition, AdjVertex dstVertexPosition) {
    this.srcVertexPosition = srcVertexPosition.vertexPosition;
    this.dstVertexPosition = dstVertexPosition.vertexPosition;
    element = e;
  }

  @Override
  public E element() {
    return element;
  }

  @Override
  public boolean equals(Object o) {
    if (o != null && o.getClass() == AdjEdge.class) {
      var edge = (AdjEdge) o;
      return (edge.srcVertexPosition.element() == srcVertexPosition.element() && edge.dstVertexPosition.element() == dstVertexPosition.element()) || (edge.srcVertexPosition.element() == dstVertexPosition.element() && edge.dstVertexPosition.element() == srcVertexPosition.element());
    }
    return false;
  }

  @Override
  public String toString() {
    return "{element:" + element + ", src:" + srcVertexPosition.element() + ", dst:" + dstVertexPosition.element() + "}";
  }
}

// storage of vertex
private final IList<IVertex<V>> vertices = new DLinkedList<>();
// storage of edges
private final IList<IEdge<E>> edges = new DLinkedList<>();

/**
 * Find the vertices that are connected by a given Edge.
 *
 * @param e An edge.
 * @return An array (of length 2) containing the two end vertices of {@code e}.
 * In case of error, return an array of two {@code null} elements.
 */
@Override
public IVertex<V>[] endVertices(IEdge<E> e) {
  // convert e to AdjEdge
  var edge = (AdjEdge) e;

  // create new array of length 2 that will contain
  // the edge's end vertices
  @SuppressWarnings("unchecked") IVertex<V>[] endpoints = new IVertex[2];

  // check if edge is valid
  if (invalidEdge(edge)) {
    endpoints[0] = endpoints[1] = null;
    return endpoints;
  }

  // fill array
  endpoints[0] = edge.srcVertexPosition.element();
  endpoints[1] = edge.dstVertexPosition.element();
  return endpoints;
}

/**
 * Find the Vertex that is opposite {@code v} if traveling along edge {@code e}.
 * In other words, {@code e} connects {@code v} to which other vertex?
 *
 * @param v The vertex to begin at.
 * @param e The edge to travel along.
 * @return The vertex opposite {@code v} along edge {@code e}. In case of error,
 * return {@code null}.
 */
@Override
public IVertex<V> opposite(IVertex<V> v, IEdge<E> e) {
  // convert v and e to AdjVertex and AdjEdge
  var edge = (AdjEdge) e;
  var vertex = (AdjVertex) v;

  // sanity check
  if (invalidEdge(edge) || invalidVertex(vertex)) return null;

  // check relationship between vertex and edge
  if (edge.srcVertexPosition.element() == vertex) {
    return edge.dstVertexPosition.element();
  } else if (edge.dstVertexPosition.element() == vertex) {
    return edge.srcVertexPosition.element();
  }
  return null;
}

/**
 * Find whether two vertices are adjacent. Two vertices are adjacent if
 * there is an edge that connects them directly.
 *
 * @param v The first vertex to check.
 * @param w The second vertex to check.
 * @return {@code true} if {@code v} and {@code w} are connected by an edge,
 * {@code false} otherwise.
 */
@Override
public boolean areAdjacent(IVertex<V> v, IVertex<V> w) {
  // convert v and w to AdjVertex
  var src = (AdjVertex) v;
  var dst = (AdjVertex) w;

  // sanity check
  if (invalidVertex(src) || invalidVertex(dst)) return false;

  var it = src.edgePosition.iterator();
  while (it.hasNext()) {
    var edge = (AdjEdge) it.next().element();
    if (edge.srcVertexPosition.element() == dst || edge.dstVertexPosition.element() == dst) {
      return true;
    }
  }
  return false;
}

/**
 * Replace the element of vertex {@code v} with a new element ({@code o}).
 *
 * @param v The vertex whose element should be changed.
 * @param o The new element to store at this vertex.
 * @return The old element that was stored in {@code v} before this method was called.
 * If {@code v} is not valid, return {@code null}.
 */
@Override
public V replace(IVertex<V> v, V o) {
  var vertex = (AdjVertex) v;
  if (invalidVertex(vertex) || o == null) {
    return null;
  }
  var old = vertex.element;
  vertex.element = o;
  return old;
}

/**
 * Replace the element of edge {@code e} with a new element ({@code o}).
 *
 * @param e The edge whose element should be changed.
 * @param o The new element to store at this edge.
 * @return The old element that was stored in {@code e} before this method was called.
 * If {@code e} is not valid, return {@code null}.
 */
@Override
public E replace(IEdge<E> e, E o) {
  var edge = (AdjEdge) e;
  if (invalidEdge(edge) || o == null) {
    return null;
  }
  var old = edge.element;
  edge.element = o;
  return old;
}

/**
 * Insert a new vertex into the graph. The element in the new vertex is given as parameter {@code o}.
 *
 * @param o The element to be stored in the new vertex.
 * @return The vertex that was created. If the vertex already exists, return the existing vertex.
 */
@Override
public IVertex<V> insertVertex(V o) {
  // we will not allow null vertex
  if (o == null) {
    return null;
  }
  var it = vertices.iterator();
  while (it.hasNext()) {
    var tmp = it.next();
    if (tmp.element() == o) {
      return tmp;
    }
  }
  var vertex = new AdjVertex(o);
  vertex.vertexPosition = vertices.insertLast(vertex);
  return vertex;
}

/**
 * Insert a new edge into the graph. The edge should connect the vertices {@code v} and {@code w},
 * and have element {@code o}.
 *
 * @param v The first vertex to connect.
 * @param w The second vertex to connect.
 * @param o The element to store in this edge.
 * @return The new edge that is created. If {@code v} or {@code w} is not valid, then return null.
 */
@Override
public IEdge<E> insertEdge(IVertex<V> v, IVertex<V> w, E o) {
  var src = (AdjVertex) v;
  var dst = (AdjVertex) w;

  // sanity check
  if (invalidVertex(src) || invalidVertex(dst) || o == null) {
    return null;
  }

  // self loop edge is not allowed
  if (v == w) {
    throw new RuntimeException("self loop edge is not allowed");
  }

  // find if edge already exists
  var edge = new AdjEdge(o, src, dst);
  var it = src.edgePosition.iterator();
  while (it.hasNext()) {
    var tmp = it.next().element();
    if (tmp.equals(edge)) {
      throw new RuntimeException("parallel edge is not allowed");
    }
  }

  // add to edge list
  edge.edgeIPosition = edges.insertLast(edge);

  // add current edge to src and dst vertex
  edge.meInSrcListPosition = src.edgePosition.insertLast(edge.edgeIPosition);
  edge.meInDstListPosition = dst.edgePosition.insertLast(edge.edgeIPosition);
  return edge;
}

/**
 * Remove a vertex from the graph.
 *
 * @param v The vertex to be removed.
 * @return The element that was stored at that vertex before it was removed.
 * If {@code v} is not valid, return {@code null}.
 */
@Override
public V removeVertex(IVertex<V> v) {
  var vertex = (AdjVertex) v;

  // sanity check
  if (invalidVertex(vertex)) {
    return null;
  }

  var adjVertex = (AdjVertex) vertices.remove(vertex.vertexPosition);
  assert adjVertex == vertex;

  // iterate over removed value and remove related edge
  var it = adjVertex.edgePosition.iterator();
  while (it.hasNext()) {
    // an edge that connected to this vertex
    var edge = (AdjEdge) it.next().element();
    if (edge == null) continue;
    removeEdge(edge);
  }
  return adjVertex.element();
}

/**
 * Remove an edge from the graph.
 *
 * @param e The edge to be removed.
 * @return The element that was stored at that edge before it was removed.
 */
@Override
public E removeEdge(IEdge<E> e) {
  // convert e to AdjEdge
  var edge = (AdjEdge) e;

  if (invalidEdge(edge)) {
    return null;
  }
  return removeEdge(edge);
}

/**
 * Find the edges that are incident on {@code v}. This is should be an iterator that can iterate
 * over all the edges that are connected to vertex {@code v}.
 *
 * @param v The vertex that the edges should be connected to.
 * @return An iterator that can iterate over all the edges connected to {@code v}.
 */
@Override
public IIterator<IEdge<E>> incidentEdges(IVertex<V> v) {
  var vertex = (AdjVertex) v;
  IList<IEdge<E>> ret = new DLinkedList<>();

  // return an empty iterator if vertex is not valid
  if (invalidVertex(vertex)) {
    return ret.iterator();
  }

  var it = vertex.edgePosition.iterator();
  while (it.hasNext()) {
    ret.insertLast(it.next().element());
  }
  return ret.iterator();
}

/**
 * Find all the vertices in the graph.
 *
 * @return An iterator that can iterate over all the vertices in the graph.
 */
@Override
public IIterator<IVertex<V>> vertices() {
  return vertices.iterator();
}

/**
 * Find all the edges in the graph.
 *
 * @return An iterator that can iterate over all the edges in the graph.
 */
@Override
public IIterator<IEdge<E>> edges() {
  return edges.iterator();
}

// helper function to get vertex count
public int VertexCount() {
  return vertices.size();
}

// helper function to get edge count
public int EdgeCount() {
  return edges.size();
}

private E removeEdge(AdjEdge edge) {
  // remove from src vertex
  var src = (AdjVertex) edge.srcVertexPosition.element();
  assert src.edgePosition.remove(edge.meInSrcListPosition).element() == edge;

  // remove from dst vertex
  var dst = (AdjVertex) edge.dstVertexPosition.element();
  assert dst.edgePosition.remove(edge.meInDstListPosition).element() == edge;

  var adjEdge = (AdjEdge) edges.remove(edge.edgeIPosition);
  assert adjEdge == edge;

  return edge.element();
}

// helper function to print graph
@Override
public String toString() {
  var sb = new StringBuilder();
  sb.append("adjacent list:\n");
  var vertexIIterator = vertices.iterator();
  while (vertexIIterator.hasNext()) {
    var vertex = (AdjVertex) vertexIIterator.next();
    sb.append("vertex ");
    sb.append(vertex.element());
    sb.append(", connected edges: ");
    sb.append("[");
    var edgeIIterator = vertex.edgePosition.iterator();
    var sep = "";
    while (edgeIIterator.hasNext()) {
      sb.append(sep);
      sep = ",";
      sb.append(edgeIIterator.next().element().element());
    }
    sb.append("]\n");
  }
  return sb.toString();
}

// helper function to print graph in dot format
// you can use `graphviz` to visualize the output
// `sudo apt install graphviz`
// `dot -Tpng -o graph.png graph.dot`
public String toDot() {
  var sb = new StringBuilder();
  sb.append("graph G {\n");
  // output vertex
  {
    var it = vertices.iterator();
    while (it.hasNext()) {
      var vertex = (AdjVertex) it.next();
      sb.append(vertex.element());
      sb.append(";\n");
    }
  }
  // output edges
  {
    var it = edges.iterator();
    while (it.hasNext()) {
      var edge = (AdjEdge) it.next();
      var src = edge.srcVertexPosition.element();
      var dst = edge.dstVertexPosition.element();
      sb.append(src.element());
      sb.append(" -- ");
      sb.append(dst.element());
      sb.append(" [label=\"");
      sb.append(edge.element());
      sb.append("\"]");
      sb.append(";\n");
    }
  }
  sb.append("}\n");
  return sb.toString();
}

private boolean invalidEdge(AdjEdge edge) {
  if (edge == null) return true;
  var it = edges.iterator();
  while (it.hasNext()) {
    var tmp = (AdjEdge) it.next();
    if (tmp.equals(edge)) {
      return false;
    }
  }
  return true;
}

private boolean invalidVertex(AdjVertex vertex) {
  var it = vertices.iterator();
  while (it.hasNext()) {
    var tmp = (AdjVertex) it.next();
    if (tmp.equals(vertex)) {
      return false;
    }
  }
  return true;
}
}
# 1
