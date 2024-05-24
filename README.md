package graph.impl;

import graph.core.*;
import graph.util.DLinkedList;


public class AdjacencyListGraph<V, E> implements IGraph<V, E> {
/**
 * Internal class to store vertex.
 */
class AdjacentListVertex implements IVertex<V> {
  // element stored in vertex
  V element;
  // position in list `vertices`
  IPosition<IVertex<V>> vertexPosition;
  // list of edges connected to this vertex
  IList<IPosition<IEdge<E>>> edgePosition = new DLinkedList<>();

  // constructor
  public AdjacentListVertex(V v) {
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
    if (o != null && o.getClass() == AdjacentListVertex.class) {
      var vertex = (AdjacentListVertex) o;
      return vertex.element == element;
    }
    return false;
  }
}

/**
 * Internal class to store position of vertex used by Edge.
 * It is used to store the position of vertex in vertex list and
 * current edge's position in affiliated vertex list's edge list.
 * Link relationship between vertex and edge like this:
 * If there is an edge between V1 and V2 linked by E1, then
 * 1. V1 will have E1 in its edge list and
 * 2. V2 will have E1 in its edge list and
 * 3. E1 will have V1 and V2 in its src and dst respectively.
 */
class Position {
  // Vertex position in vertex list
  IPosition<IVertex<V>> vertexPosition;
  // Edge position in vertex affiliated list
  IPosition<IPosition<IEdge<E>>> edgePosition;
}

/**
 * Internal class to store edge.
 */
class AdjacentListEdge implements IEdge<E> {
  E element;
  // position in list `edges`
  IPosition<IEdge<E>> edgeIPosition;

  Position src = new Position();
  Position dst = new Position();

  // constructor
  AdjacentListEdge(E e, AdjacentListVertex srcVertexPosition, AdjacentListVertex dstVertexPosition) {
    this.src.vertexPosition = srcVertexPosition.vertexPosition;
    this.dst.vertexPosition = dstVertexPosition.vertexPosition;
    element = e;
  }

  @Override
  public E element() {
    return element;
  }

  @Override
  public boolean equals(Object o) {
    if (o != null && o.getClass() == AdjacentListEdge.class) {
      var edge = (AdjacentListEdge) o;
      return (edge.src.vertexPosition.element() == src.vertexPosition.element()
          && edge.dst.vertexPosition.element() == dst.vertexPosition.element())
          || (edge.src.vertexPosition.element() == dst.vertexPosition.element()
          && edge.dst.vertexPosition.element() == src.vertexPosition.element());
    }
    return false;
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
 */
@Override
public IVertex<V>[] endVertices(IEdge<E> e) {
  // convert e to AdjEdge
  var edge = (AdjacentListEdge) e;

  // create new array of length 2 that will contain
  // the edge's end vertices
  @SuppressWarnings("unchecked") IVertex<V>[] endpoints = new IVertex[2];

  // check if edge is valid
  if (edgeError(edge)) {
    endpoints[0] = endpoints[1] = null;
    return endpoints;
  }

  // fill array
  endpoints[0] = edge.src.vertexPosition.element();
  endpoints[1] = edge.dst.vertexPosition.element();
  return endpoints;
}

/**
 * Find the Vertex that is opposite {@code v} if traveling along edge {@code e}.
 * In other words, {@code e} connects {@code v} to which other vertex?
 *
 * @param v The vertex to begin at.
 * @param e The edge to travel along.
 * @return The vertex opposite {@code v} along edge {@code e}.
 */
@Override
public IVertex<V> opposite(IVertex<V> v, IEdge<E> e) {
  // convert v and e to AdjVertex and AdjEdge
  var edge = (AdjacentListEdge) e;
  var vertex = (AdjacentListVertex) v;

  // sanity check
  if (edgeError(edge) || vertexError(vertex)) return null;

  // check relationship between vertex and edge
  if (edge.src.vertexPosition.element() == vertex) {
    return edge.dst.vertexPosition.element();
  } else if (edge.dst.vertexPosition.element() == vertex) {
    return edge.src.vertexPosition.element();
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
  var src = (AdjacentListVertex) v;
  var dst = (AdjacentListVertex) w;

  // sanity check
  if (vertexError(src) || vertexError(dst)) return false;

  var curr = src.edgePosition.iterator();
  while (curr.hasNext()) {
    var edge = (AdjacentListEdge) curr.next().element();
    if (edge.src.vertexPosition.element() == dst || edge.dst.vertexPosition.element() == dst) {
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
 */
@Override
public V replace(IVertex<V> v, V o) {
  if (o == null) return null;

  var vertex = (AdjacentListVertex) v;
  if (vertexError(vertex)) {
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
 */
@Override
public E replace(IEdge<E> e, E o) {
  if (o == null) return null;

  var edge = (AdjacentListEdge) e;
  if (edgeError(edge)) {
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
 * @return The vertex that was created.
 */
@Override
public IVertex<V> insertVertex(V o) {
  // we will not allow null vertex
  if (o == null) {
    return null;
  }
  var curr = vertices.iterator();
  while (curr.hasNext()) {
    var tmp = curr.next();
    if (tmp.element() == o) {
      return tmp;
    }
  }
  var vertex = new AdjacentListVertex(o);
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
 * @return The new edge that is created.
 */
@Override
public IEdge<E> insertEdge(IVertex<V> v, IVertex<V> w, E o) {
  if (o == null) return null;

  var src = (AdjacentListVertex) v;
  var dst = (AdjacentListVertex) w;

  // sanity check
  if (vertexError(src) || vertexError(dst)) {
    return null;
  }

  // self loop edge is not allowed
  if (v == w) {
    throw new RuntimeException("self loop edge found");
  }

  // find if edge already exists
  var edge = new AdjacentListEdge(o, src, dst);
  var curr = src.edgePosition.iterator();
  while (curr.hasNext()) {
    var tmp = curr.next();
    if (tmp.element().equals(edge)) {
      throw new RuntimeException("parallel edge found");
    }
  }

  // add to edge list
  edge.edgeIPosition = edges.insertLast(edge);
  edge.src.edgePosition = src.edgePosition.insertLast(edge.edgeIPosition);
  edge.dst.edgePosition = dst.edgePosition.insertLast(edge.edgeIPosition);
  return edge;
}

/**
 * Remove a vertex from the graph.
 *
 * @param v The vertex to be removed.
 * @return The element that was stored at that vertex before it was removed.
 */
@Override
public V removeVertex(IVertex<V> v) {
  var vertex = (AdjacentListVertex) v;

  // sanity check
  if (vertexError(vertex)) {
    return null;
  }

  var adjVertex = (AdjacentListVertex) vertices.remove(vertex.vertexPosition);
  assert adjVertex == vertex;

  // iterate over removed value and remove related edge
  var curr = adjVertex.edgePosition.iterator();
  while (curr.hasNext()) {
    // an edge that connected to this vertex
    var edge = (AdjacentListEdge) curr.next().element();
    if (edge != null) deleteEdge(edge);
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
  var edge = (AdjacentListEdge) e;

  if (edgeError(edge)) {
    return null;
  }
  return deleteEdge(edge);
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
  var vertex = (AdjacentListVertex) v;
  IList<IEdge<E>> result = new DLinkedList<>();

  if (vertexError(vertex)) {
    return result.iterator();
  }

  var it = vertex.edgePosition.iterator();
  while (it.hasNext()) {
    result.insertLast(it.next().element());
  }
  return result.iterator();
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

public int VertexCount() {
  return vertices.size();
}

public int EdgeCount() {
  return edges.size();
}

private E deleteEdge(AdjacentListEdge edge) {
  // remove from src vertex
  var src = (AdjacentListVertex) edge.src.vertexPosition.element();
  var srcElem = src.edgePosition.remove(edge.src.edgePosition).element();
  assert srcElem == edge;

  // remove from dst vertex
  var dst = (AdjacentListVertex) edge.dst.vertexPosition.element();
  var dstElem = dst.edgePosition.remove(edge.dst.edgePosition).element();
  assert dstElem == edge;

  var removed = (AdjacentListEdge) edges.remove(edge.edgeIPosition);
  assert removed == edge;
  return edge.element();
}


@Override
public String toString() {
  var sb = new StringBuilder();
  sb.append("Adjacency List:\n");
  var vertexIIterator = vertices.iterator();
  while (vertexIIterator.hasNext()) {
    var head =(AdjacentListVertex) vertexIIterator.next();
    sb.append("Vertex ");
    sb.append(head.element());
    sb.append(": [");

    var edge = head.edgePosition.iterator();
    var sep = "";
    while (edge.hasNext()) {
      sb.append(sep);
      sb.append(edge.next().element().element());
      sep = ", ";
    }
    sb.append("]\n");
  }
  return sb.toString();
}

private boolean edgeError(AdjacentListEdge edge) {
  if (edge == null) return true;
  var curr = edges.iterator();
  while (curr.hasNext()) {
    var tmp = (AdjacentListEdge) curr.next();
    if (tmp.equals(edge)) {
      return false;
    }
  }
  return true;
}

private boolean vertexError(AdjacentListVertex vertex) {
  var curr = vertices.iterator();
  while (curr.hasNext()) {
    var tmp = (AdjacentListVertex) curr.next();
    if (tmp.equals(vertex)) {
      return false;
    }
  }
  return true;
}
}
